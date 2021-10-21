# Nodejs Address already in use

1. macOS 및 Linux에서 포트를 사용하는 프로세스 찾기

   - `lsof -i TCP:<포트번호>`

     ex) `lsof -i TCP:3030`

2. pid 확인한 후 `kill -9 <pid>` 명령어 수행



