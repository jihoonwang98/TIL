# [Spring MVC] Validation 객체 정리





![](https://docs.google.com/drawings/d/e/2PACX-1vRZysllWM2T7GyA1qf2yWICZXWKs5fNs0Rzc_cSy189qSX3xfjeUS0eBn4olRlhNGn4n1YC3GVOiRJ0/pub?w=1417&h=736)



## `Errors` 인터페이스

특정 객체의 data-binding, validation 에러에 관한 정보들을 저장하고 보여주는 메서드들을 정의한 인터페이스.

필드명은 





주의: `Errors` 객체들은 single-threaded 객체들이다.















