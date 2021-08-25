# Tistory 클론 미니프로젝트



### API

| 요청                       | 설명                  | 반환                    |
| -------------------------- | --------------------- | ----------------------- |
| GET /api/v1/boards         | 모든 boards 가져오기  | List\<BoardDTO> boards; |
| GET /api/v1/boards/{id}    | board detail 가져오기 | BoardDTO board;         |
| POST /api/v1/boards        | board 생성하기        |                         |
| PATCH /api/v1/boards/{id}  | board 수정하기        |                         |
| DELETE /api/v1/boards/{id} | board 삭제하기        |                         |
|                            |                       |                         |



### Controller

| 요청                         | 설명                                                         | View              |
| ---------------------------- | ------------------------------------------------------------ | ----------------- |
| GET /, GET /category         | 메인 페이지                                                  | main.html         |
| GET /{boardId}               |                                                              | boardDetails.html |
| GET /modify/{boardId}        |                                                              | boardModify.html  |
| GET /category/{categoryName} | 메인 페이진데 내용물이 해당 카테고리                         | main.html         |
| GET /tag                     | 메인 페이진데 내용물이 모든 태그                             | main.html         |
| GET /tag/{tagName}           | 메인 페이진데 내용물이 해당 태그                             | main.html         |
| GET /{boardId}#comment121323 | 최신 댓글 클릭하면 해당 게시물로 가서 댓글창이 바로 보이게 함 | boardDetails.html |
| GET /guestbook               | 메인 페이진데 방명록 작성 부분이 나옴                        | main.html         |













