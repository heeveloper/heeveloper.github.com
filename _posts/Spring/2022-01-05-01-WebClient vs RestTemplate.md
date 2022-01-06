---
layout: post
title:  "WebClient vs RestTemplate"
date:   2022-01-05 23:10:00
author: Heeveloper
categories: Spring
tags: Spring Springboot WebClient RestTemplate
excerpt: Spring
mathjax : false
---
{% include adsense_content.html %}

* content
{:toc}


Spring에서 제공하는 대표적인 Http 통신 템플릿으로  RestTemplate과 WebClient 가 있다.  
자세한 설명은 [https://www.baeldung.com/spring-webclient-resttemplate](https://www.baeldung.com/spring-webclient-resttemplate)에서 확인할 수 있으며,  
정리하자면 아래와 같다.
<br>
<br>

> 1. WebClient가 RestTemplate 이후에 나왔다. (Spring 5.0부터 적용)
> 2. RestTemplate은 sync blocking client로서 Response가 올 때까지 다음 행동으로 넘어갈 수 없다.
> 3. WebClient는 asynchronous, non-blocking client
> 4. RestTemplate은 곧 Deprecated 된다. (참고 : [Java Doc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html))

<br>

**결론**  

non-blocking 접근방식이 상대적으로 더 적은 System Resource를 소모하기 때문에 **WebClient**를 사용하자!


