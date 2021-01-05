# OAuth

인터넷 사용자들이 비밀번호를 제공하지 않고 다른 웹사이트 상의 자신들의 정보에 대해 웹사이트나 애플리케이션의 접근 권한을 부여할 수 있는 공통적인 수단으로써 사용되는 접근 위임을 위한 개방형 표준이다. 이 매커니즘은 여러 기업들에 의해 사용되는데, 이를테면 아마존, 구글, 트워터 등이 있으며 사용자들이 타사 애플리케이션이나 웹사이트의 계정에 관한 정보를 공유할 수 있게 허용한다.

## 작동원리

1. 유저가 로그인 버튼 클릭
2. 그 링크는 유저를 view로 이동시키는데 그 view는 깃헙으로 redirect시킨다.
3. 유저가 확인 버튼을 누르면 github은 다시 유저를 지정한 redirect url로 리디렉션 시킨다.

## github login 정리

    1. Github Login 버튼 클릭
    2. 그 링크는 github.com으로 redirect됨
    3. github.com 은 몇몇 데이터들을 필요로함 (client_id, redirect_uri, scope)
    4. user가 github에서 accept버튼을 누르면 다시 redirect_uri로 redirect 됨 Github_callback의 결과값
    5. github_callback에서는 url에서 code를 가져옴
    6. token을 액세스 하기 위해서 코드를가지고 request함
    7. 그럼 json data를 받을 수 있음 "Accept": "application/json" << 이 라인에 의해서 access_token이 들어있음
    8. access_token를 가지고 github api에 request할 수 있음 headers에 token을 보냄
    9. 결과적으로 user의 profile을 얻을 수 있음 json 형태로
