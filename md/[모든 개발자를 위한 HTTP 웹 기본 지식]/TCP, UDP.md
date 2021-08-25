## TCP, UDP



### 인터넷 프로토콜 스택의 4계층



![](https://lh5.googleusercontent.com/dC3y3nwYaxgWLJa6y_1o-Jn-A-C1soeEdPtCe-M9T0xyAACAIDMXBtz9XaXNkBaMeooquxOkMv5TGkthYxbjMBpj1uBcgOCvvHqiyEWoGDIN-lbSmFTM_cb5Zg1eGnkc-76TZ6gG)

**IP**위에 **TCP**를 올려서 보완해준다고 생각하면 된다.





### 프로토콜 계층

![](https://lh6.googleusercontent.com/XGTmx5WfuT-rMODF2MKb2yxMIdf7NxOZ0c3PNPOQCWV5PyoLxZzHsFhmpfWnQwhJftcO0qjt18DqxmZwnsvdmLl8YWpgOkSK6cUq7iRgUEANsOwCITgcLMn9LNrtMwHMGnsgLbuR)



미국에 있는 친구한테 Hello, world! 전달하는 과정

![](https://lh5.googleusercontent.com/MUSs6pfwDtTLwf4SKtIrU8-YRbLJ2dWUip-mF0Cc0NzNKozCXlLmMNXHWSguuXYfFAyDpyZRdfIx9miPrRexK_a3fN2iUyTEGKjkPDGszsQQ_bWSO0Qy9b40RMhEXCQDk9EmKubk)





![](https://lh3.googleusercontent.com/KkMfZTYmQ-RdU0PL7Fvv10dSfTRYY7tBA3PQJyzAUPs9r4AufCoRiVaRDoEqm_2QT1m3ezSYjeflSNd3SUUXZxUu00oNwj1jiRtqoBFtG9gy1L2qpUCoUI4fsNufJ4a0WrlJJcYf)



**PORT, 전송 제어, 순서, 검증 정보** 등이 추가된다.





### TCP 특징

전송 제어 프로토콜(**Transmission Control Protocol**)



- **연결지향 - TCP 3 way handshake (가상 연결)**
  - IP의 **비연결성** 해결 -> 패킷을 받을 대상이 서비스 가능해야만 전달함.
- **데이터 전달 보증, 순서 보장**
  - IP의 **비신뢰성** 해결 -> **패킷 누락, 패킷 순서** 보장해줌.



- **신뢰할 수 있는 프로토콜**
- **현재는 대부분 TCP 사용**





![](https://lh4.googleusercontent.com/zYoEP79fnz3wmz02BcfuHESKaY0vZtxpuMuWFjwK1EwNw-8TW9xvC3kqu6n-Ym1s_FioP_5BMlj0yPgACkQjSztfkDjI5JMr2ENNbD1jXQdanrQiDAACOffZDQujlsXPAePNhF10)



**(1)** **SYN**(synchronize): 클라이언트가 서버에게 **SYN** 요청

**(2)** **SYN + ACK**(acknowledge): 서버가 클라이언트에게 **SYN**에 대한 **ACK**와 함께 **SYN** 요청

**(3)** **ACK**: 클라이언트가 서버에게 **SYN**에 대한 **ACK** 전송 

**(4)** **데이터 전송**





만약 처음에 **서버가 꺼져있었다면?**

클라이언트가 서버에 **SYN**을 보냈을 때, 응답이 없어서 연결이 안된다.







### 데이터 전달 보증

![](https://lh3.googleusercontent.com/G9aGhbwUdNOiwlrergFWD2LoatYFLxCW-noT6qROa-SssOduyq00mao1jZ_ukKvjlvJNOrhAJh8MeboSiB2y_maXs8Ri3nqcX27T4jyf8dh93WSUMMTHNvwyVOrVYuxNykI8Bi92)





### 순서 보장

![](https://lh6.googleusercontent.com/l_yqk_XRVnqu6_8PJi0ATp08OFy-bbvay9laef_plVa3Yl05YLdsJRzbe3W48kyr1Kd-_8kKPTi-WTI6XxdgrX7Fpiy1caQiFuZii94IVNqyRV6GoMZh5MJteDi3JKul2q7kJ8x6)





### UDP 특징

사용자 데이터그램 프로토콜(**User Datagram Protocol**)



- **하얀 도화지에 비유(기능이 거의 없음)**

- **연결지향 - TCP 3 way handshake X**

- **데이터 전달 보증 X**

- **순서 보장 X**

- **데이터 전달 및 순서가 보장되지 않지만, 단순하고 빠름**

  (TCP는 3 way handshake랑 데이터 전달 보증할 때 거치는 과정이 더 많음)



- **정리**
  - **IP와 거의 같다. +PORT +체크섬(데이터 잘 왔는지 검증해주는 데이터) 정도만 추가**
  - **애플리케이션에서 추가 작업 필요**