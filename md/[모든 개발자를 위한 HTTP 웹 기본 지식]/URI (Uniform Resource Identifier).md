## URI (Uniform Resource Identifier)



### **UR**<span style="color:red">I</span>? **UR**<span style="color:red">L</span>? **UR**<span style="color:red">N</span>? 차이???

![](https://lh3.googleusercontent.com/KdLuoa5K_hI_Uo8XQ8m_aIk6GUcQArkJcZM0vjSR6qGHAbOmwBtakYAa-_19J1oyVIFm8l8_crUW55v0lXYjMcHN-Jel1DmefqN42Wy6bC3dr8aO4XVucN2s1_34ZcD8rizujhLS)





![](https://lh3.googleusercontent.com/0n1KQhAWfbPMfYydlobTRlNwzYIeTzkWpcIXeuE2HMbuLDDiFdopAGBpnWsX8MG_iW12vWOx_5U2LkGmGGGLviMNgNJzRP-7fJ43T6zTAWbNO23qbSJWdvh_jE2l7fZ32ARARxRZ)







### URI 단어 뜻

- **U**niform: 리소스를 식별하는 통일된 방식
- **R**esource: 자원, URI로 식별할 수 있는 모든 것(제한 없음)
- **I**dentifier: 다른 항목과 구분하는데 필요한 정보



- **URL**: Uniform Resource Locator
- **URN**: Uniform Resource Name





### URL, URN

- URL - Locator: 리소스가 있는 위치를 지정
- URN - Name: 리소스에 이름을 부여
- 위치는 변할 수 있지만, 이름은 변하지 않는다.
- urn:isbn:8960777331 (어떤 책의 isbn URN)
- URN 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화되지 않음
- 앞으로 URI를 URL과 같은 의미로 이야기하겠음







### URL

- `scheme://[userinfo@]host[:port][/path][?query][#fragment]`
- `https://www.google.com:443/search?q=hello&hl=ko`



- 프로토콜(`https`)
- 호스트명(`www.google.com`)
- 포트 번호(`443`)
- path(`/search`)
- 쿼리 파라미터(`q=hello&hl=ko`)





### URL - scheme

- 주로 프로토콜 사용
- 프로토콜: 어떤 방식으로 자원에 접근할 것인가 하는 약속 규칙 (예: http, https, ftp 등등)
- http는 80 포트, https는 443 포트를 주로 사용, 포트는 생략 가능
- https는 http에 보안 추가 (HTTP Secure)





### URL - userinfo

- URL에 사용자정보를 포함해서 인증
- 거의 사용하지 않음





### URL - host

- 호스트명
- 도메인명 또는 IP주소를 직접 사용가능





### URL - PORT

- 포트(PORT)
- 접속 포트
- 일반적으로 생략, 생략시 http는 80, https는 443





### URL - path

- 리소스 경로(path), 계층적 구조





### URL - query

- key = value 형태
- ?로 시작, &로 추가 가능 `?keyA=valueA&keyB=valueB`
- query parameter, query string 등으로 불림, 웹 서버에 제공하는 파라미터, 문자 형태





### URL - fragment

- fragment
- html 내부 북마크 등에 사용
- 서버에 전송하는 정보 아님









> 본 포스팅은 인프런의 **모든 개발자를 위한 HTTP 웹 기본 지식 by 김영한** 강좌를 듣고 정리한 내용입니다. 
>
> 링크: https://inf.run/tHdt