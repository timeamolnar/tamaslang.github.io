---
layout: post
title: Spring with Java 8 Optional
---

Java 8 introduced a new keyword: <a href="http://docs.oracle.com/javase/8/docs/api/java/util/Optional.html" target="_blank">Optional</a>
to handle the well known null references problem. A good explanation about using optional is available
<a href="http://examples.javacodegeeks.com/core-java/util/optional/java-8-optional-example/" target="_blank">here</a>.

It might be a cleaner way than plain null-checks, but in fact it didn't convince me as there is a major issue with load your code full with Optional-s, because:

<strong>You need to change your API and that will affect everybody using it, because</strong>
<blockquote><p>
  It is not possible to write a method that takes either an X or an Optional&lt;X&gt;.
</p></blockquote>

See other aspects in a detailed criticism: <a href="https://www.voxxed.com/blog/2015/01/embracing-void-6-refined-tricks-dealing-nulls-java/" target="_blank">Do not use Optional</a>

So I decided <strong>not to use</strong> Optional in any of my code, but then I came across a
<a href="http://spring.io/blog/2015/01/14/springone2gx-2014-replay-spring-framework-on-java-8" target="_blank">webinar</a>
presented on SpringOne 2014 about Spring framework on Java8. <br/>
Combined with Spring I found 2 useful places where applying optional make sense,
but mainly because a framework handles it for me.

###Example:
In the following example 2 possible usages are shown. One is to deal with an optional @RequestParam (or @PathVariable)
the other it to deal with an optional service.


```java
@RestController
@RequestMapping("/api/masterdata")
public class CountryResourceImpl implements CountryResource {

    // instead of @Autowired(required = false)
    @Autowired Optional<NotificationService> notificationService;

    @Override
    @RequestMapping(value = "/countries",
        method = RequestMethod.GET,
        produces = "application/json; charset=utf-8")
    @ResponseStatus(HttpStatus.OK)
    // instead of: @RequestParam(required = false) String lang
    public List<CountryDataDTO> getAllCountriesInLanguage(@RequestParam Optional<String> lang) {
        // optional with fallback to default
        String langValue = lang.orElse(DEFAULT_LANG);

        // notification service, notify if service is present
        notificationService.ifPresent((service) -> {service.eventOccured("...");} );  
    }
}
```

After all using Optional can be convenient when Spring translates X to Optional&lt;X&gt; for me.<br/>
Your code can also be shorter and cleaner with using lambda expressions in ifPresent parameter.

T.