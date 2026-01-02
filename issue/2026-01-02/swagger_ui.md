# swagger ui

## 로컬에서 swagger ui 로 테스트할 때 이슈

- 이전 회사에서는 백엔드 개발자에게 CORS 이슈가 있으니 CORS 헤더에 추가해달라고 요청
- 외부 기업과 함께 하는 개발이고, 파일로 swagger 를 전달 받은게 처음이라 구글링 해보니 `http-server` 로 실행할 것을 추천해서 실행 -> `--cors` 도 추가해보고 프록시 서버도 구축해봤지만 안되어서 결국 CORS 추가를 요청
- 결론적으로는 postman으로 실행하게 됨(postman으로 요청하면 브라우저에서 CORS 이슈가 발생하지 않는다고 함)

## Issue: CORS problem when testing Swagger UI locally

When testing the API using Swagger UI in a local environment, I encountered CORS-related issues.

- In my previous company, when similar issues occurred, we usually asked backend developers to add the required CORS headers to the server configuration.
- In this project, I am collaborating with an external company, and it was my first time receiving Swagger UI as a static file. After some research, I found recommendations to run it using tools like `http-server`.
- I tried running Swagger UI with `http-server`, added the `-cors` option, and even set up a proxy server, but the CORS issue persisted.
- Eventually, I requested the backend team to add the necessary CORS headers.

**Conclusion:**

I ended up using Postman to test the API. Since requests made from Postman are not subject to browser CORS restrictions, the API calls worked without any issues.