# 컨트롤러의 원하는 파라미터만 변환하기

컨트롤러에 인자(argument) 들어오는 문자열 값이 있습니다. 그 인자를 저장하는 변수의 이름은 date이고, 값은 (정상적으로 들어온다면) 8자리 문자열로 이루어져있습니다.

```
@GetMapping(value = "/spaces/{spaceNo}/timeslots", produces = MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<List<SpaceBookDto>> getTimeslotOfTheDay(@PathVariable("spaceNo") Integer spaceNo
                , @RequestParam(required = false) String date) {
  ...
}
```

컨트롤러로 이 date 값이 들어오면 컨트롤러에서는 변환을 한후 service 계층에 넘겨줍니다. 그런데 이 값을 받는 controller가 여러개가, 여러 파일에 걸쳐 있습니다. date 값을 변환하는 private 메서드를 controller 별로 중복되게 작성하지 않고 관심사를 분리하려고 합니다.

## InitBinder와 WebDataBinder

@InitBinder를 통해 converter를 WebDataBinder에 등록하는 방법이 있습니다. 아래와 같이요.

```
@Controller
public class FormController {

	@InitBinder
	public void initBinder(WebDataBinder binder) {
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		dateFormat.setLenient(false);
		binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
	}

	// ...
}
```

WebDataBinder는 크게 2가지의 일을 합니다.

- request에 포함된 파라미터를 모델 객체 저장해(**binding**이라고 하죠) 반환합니다.
- request의 파라미터인 String값을 **변환**합니다.

@InitBinder가 적용된 메서드에서는 이 WebDataBinder를 인자로 받아 초기화합니다. 그래서 이 메서드에서 변환 기능을 하는 Converter나 Formatter를 WebDataBinder에 등록할 수 있습니다.

하지만 이 방법을 사용하면 @InitBinder가 적용되는 controller에 전달된 모든 String 타입의 파라미터가 변환대상이 됩니다. 어떤 예외가 발생할지 모르게 되는거죠. 그래서 커스텀 어노테이션 + HandlerMethodArgumentResolver 방식을 사용해서 제가 원하는 Controller 메서드의 파라미터에만 적용을 하려고 합니다.

## custom annotation

먼저 사용자 정의 annotation을 정의합니다.

```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface StringDate {
}

```

위의 코드에서 사용된 Java annotation들을 잠깐 살펴봅시다.
Java Annotation은 클래스, 메서드, 인스턴스 변수 등의 프로그램 요소에 메타데이터를 연결하는데 사용됩니다.

- @interface

  - java compiler에서 'StringDate'가 Java annotation 정의임을 알려주는 키워드입니다.

- @Target

  - Custom annotation을 적용할 element(method, parameter, type...)를 지정할 수 있습니다.
  - meta-annotation으로 다른 annotation(주석)에 대해 주석을 달 때만 사용할 수 있습니다. StringDate라는 주석에 Target이라는 주석을 다는 것처럼요.
  - ElementType만을 인수로 가질 수 있습니다.

- @Retention
  - 사용자 지정 annotation을 애플리케이션의 실행 중에 사용할 수 있는지, 리플렉션을 통해 검사할 수 있는지를 지정할 수 있습니다.
    - 리플렉션(Reflection)은 자바 프로그램의 내부 정보(클래스, 변수 등)를 검사하거나 수정할 수 있는 강력한 Java 프로그래밍 언어의 기능입니다.
  - 위의 코드에서는 StringDate annotation을 런타임 시 리플렉션을 통해 사용할 수 있음을 지정합니다.

## HandlerMethodArgumentResolver

`HandlerMethodArgumentResolver`는 컨트롤러 메서드로 전달되는 파라미터를 인자 값에 맞게 변환하거나 매핑하는 전략 인터페이스입니다. 이 인터페이스를 구현하여 문자열 파라미터 값을 LocalDate 타입으로 변환하는 resolver 객체를 만들겠습니다.

```
import org.springframework.core.MethodParameter;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;
import site.roombook.annotation.StringDate;

import java.time.DateTimeException;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public class StringDateArgumentResolver implements HandlerMethodArgumentResolver {

	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		return parameter.getParameterType().equals(LocalDate.class)
				&& parameter.hasParameterAnnotation(StringDate.class);
	}

	@Override
	public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
		String dateParam = webRequest.getParameter(parameter.getParameterName());
		LocalDate localDate;

		try {
			localDate = LocalDate.parse(dateParam, DateTimeFormatter.BASIC_ISO_DATE);
		} catch (DateTimeException e){
			localDate = LocalDate.now();
		}

		return localDate;
	}
}
```

`StringDateArgumentResolver#supportsParameter` 메서드에서는 resolver에서 변환하려는 파라미터인지 확인합니다. 오버라이딩 한 메서드에서는 선언된 파라미터의 타입이 LocalDate인지, custom annotation인 StringDate이 선언되어있는지를 확인합니다.

`StringDateArgumentResolver#resolveArgument` 메서드에서는 실제 변환 작업을 수행합니다.

## RequestMappingHandlerAdapter

`HandlerMethodArgumentResolver`의 구현체를 사용하기 위해서는 `RequestMappingHandlerAdapter`객체에 저장해야합니다.

`RequestMappingHandlerAdapter`는 Spring MVC 애플리케이션에 들어오는 요청을 처리하는 핵심 객체입니다. 웹 서버에 요청이 들어오면 먼저 이 HandlerAdapter 요청과 매핑된 컨트롤러 메서드를 찾아냅니다. 컨트롤러 메서드의 각 파라미터를 처리하기 위해 HandlerAdapter에 저장된 `HandlerMethodArgumentResolver` list를 돌며 파라미터를 처리할 수 있는 resolver를 찾습니다. 파라미터를 처리할 수 있는 resolver가 처리 후, 결과값을 반환하면 HandlerAdapter에 그 값을 컨트롤러 메서드에 전달합니다.

다음의 실제 구현 코드에서 resolver list를 돌며 처리할 수 있는 resolver를 찾는 과정을 확인할 수 있습니다.

```
public class HandlerMethodArgumentResolverComposite implements HandlerMethodArgumentResolver {

	private final List<HandlerMethodArgumentResolver> argumentResolvers = new ArrayList<>();
	@Nullable
	private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
		HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
		if (result == null) {
			for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) { // resolver list 돌며 확인
				if (resolver.supportsParameter(parameter)) { //변환해야하는 파라미터인지 확인
					result = resolver;
					this.argumentResolverCache.put(parameter, result);
					break;
				}
			}
		}
		return result;
	}
}
```

마지막으로 구현한 `StringDateArgumentResolver`를 등록합니다. 저는 xml 설정 파일을 사용하고 있기 때문에 아래와 같이 annotation-driven을 태그를 통해 argument-resolver를 등록했습니다.

```
<annotation-driven>
		<argument-resolvers>
				<beans:bean class="site.roombook.resolver.StringDateArgumentResolver"/>
		</argument-resolvers>
</annotation-driven>
```

끝!

# 참조

- https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-initbinder.html
- https://docs.spring.io/spring-framework/docs/5.0.2.RELEASE/kdoc-api/spring-framework/org.springframework.web.method.support/-handler-method-argument-resolver/
