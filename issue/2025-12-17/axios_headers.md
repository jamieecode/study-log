다운로드시 파일명을 서버의 응답 헤더에서 내려주는 값으로 넣으려고 했는데 axios에서 content-disposition 값을 가져오지 못하는 이슈가 있었다. 크롬 개발자 도구를 보면 응답헤더에 content-disposition이 attachment; filename*=UTF-8~~로 잘 나오고 있는데 프론트에서 콘솔로 찍어보면 나오지 않아 찾아보니 CORS 이슈라서 서버에서 핸들링해야하는 이슈였다.

When trying to set the downloaded file name using a value from the server’s response headers, I ran into an issue where Axios couldn’t access the Content-Disposition header.

In Chrome DevTools, the response headers clearly showed Content-Disposition: attachment; filename*=UTF-8...,
but when logging the headers on the frontend, Content-Disposition was missing.

After some research, I found that this was a CORS-related issue that needed to be handled on the server side.

---

Access-Control-Expose-Headers: Content-Disposition 을 넣어주면 된다.

The solution was to add the following header to the server response:
Access-Control-Expose-Headers: Content-Disposition

참고: https://ahndding.tistory.com/4
https://stackoverflow.com/questions/69160861/cant-access-content-disposition-in-axios
