# JSON Web Token

## JSON Web Token이란?

JSON Web Token은 인증 헤더 내에서 사용되는 클레임 기반 토큰 포맷이다. JSON은 웹 표준으로써 두개의 시스템 끼리 안전한 방법으로 통신할 수 있도록 설계하는 것을 도와준다.

### 지원

JWT는 C, Java, Python, C++, R, C#, PHP, Javascript, Go, Ruby, Swift 등 수많은 언어에서 지원된다.

### 자가 수용적(self-contained)

JWT는 필요한 모든 정보를 자체적으로 지니고 있다. JWT 시스템에서 발급된 토큰은 토큰에 대한 기본정보, 전달할 정보, 토큰이 검증되었다는 것을 증명해주는 signature를 포함하고 있다.

### 쉽게 전달

JWT는 자가 수용적이므로, 두 시스템 사이에서 쉽게 전달될 수 있다. 웹서버의 경우에는 HTTP 헤더에 넣어서 보낼 수 있고 url 파라미터로도 전달될 수 있다.

## JWT의 구조

JWT는 `.`를 구분자로 3가지 문자열로 정의되어 있다. 구조는 아래와 같이 이루어져있다.

![jwt](../../img/jwt.png)
**출처:https://medium.com/@ahsan.shamsudeen/jwt-signature-verification-and-beyond-2dc3143a81e5**

### Header

헤더는 두가지 정보를 가지고 있다.
typ: 토큰의 타입을 지정한다. jwt니까 `JWT` 이다.
alg: 해싱 알고리즘을 지정한다. 해싱 알고리즘은 보통 `HMAC SHA256` 또는 `RSA`가 사용되고, 이 알고리즘은 검증할 때 사용되는 signature 부분에서 사용된다.

`HMAC SHA256` 해싱알고리즘을 사용하는 jwt 토큰 헤더는 아래와 같다.

```json
{
  "typ": "JWT",
  "alg": "H256"
}
```

이 정보를 **base64**로 인코딩 하면 아래와 같다.

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9`

> JSON 형태의 객체가 base64로 인코딩 되는 과정에서 공텍과 엔터들을 사라진다.
> 따라서 아래과 같은 문자열을 인코딩한다
> `{"typ": "JWT","alg": "H256"}

### Payload

페이로드에는 토큰에 담을 정보가 들어있다. 여기에 담는 정보의 한 조각을 **Claim(클레임)** 이라고 부른다. 이 클레임은 key-value 형식으로 담겨 있다.

클레임의 종류는 크게 3가지로 분류된다.

1. Registered(등록된) claim
   등록된 클레임은 서비스에 필요한 정보가 아닌, 토큰에 대한 정보를 담기위해 정해진 클레임이다. 모든 클레임들은 선택적이다.
   - **iss**: 토큰 발급자(issuer)
   - **sub**: 토큰 제목(subject)
   - **aud**: 토큰 대상자(audience)
   - **exp**: 토큰 만료시간(expiration), 시간은 NumericDate 형식으로 되어있어야 하고 현재 시간보다 이후로 설정해야한다.
   - **nbf**: Not before의 약자로, 이 날짜가 지나기 전까지는 토큰이 처리되지 않는다.
   - **iat**: 토큰이 발급된 시간(issued at) 토큰이 얼마나 되었는지 알 수 있다.
   - **jti**: JWT의 고유 식별자로 중복적인 처리를 방지하기 위해 사용된다.
2. Public(공개) claim
   - 공개 클레임들은 충돌이 방지된 이름을 가지고 있어야 한다. 충돌을 방지하기 위해서 **URI** 형식으로 정의한다.
   ```json
   {
     "https://example.com/jwt_claims/is_admin": true
   }
   ```
3. Private(비공개) claim
   두 시스템 사이에 협의하에 사용되는 클레임 이름들이다.
   ```json
   {
     "username": "wookkl"
   }
   ```

#### Example payload

```json
{
  "iss": "example.com",
  "https://example.com/jwt_claims/is_admin": true,
  "username": "exuser1"
}
```

이 정보를 **base64**로 인코딩 하면 아래와 같다.

`eyJpc3MiOiJleGFtcGxlLmNvbSIsImh0dHBzOi8vZXhhbXBsZS5jb20vand0X2NsYWltcy9pc19hZG1pbiI6dHJ1ZSwidXNlcm5hbWUiOiJleHVzZXIxIn0=`

> base64로 인코딩할 떄 뒤에 `=` 패딩 문자를 지워주도록 한다. 그 이유는 토큰을 url 파라미터로 보낼 경우에 url-safe하지 않기 때문이다.

### Signature

이 서명에는 헤더의 인코딩 값과 페이로드의 인코딩 값을 합친구 비밀키로 해쉬하여 생성한다.

#### Pseudo code

```
data = base64urlEncode(jwt_header) + '.' + base64urlEncode(jwt_payload)
signature = HMACSHA256(data, "secretkey")
jwt_signature = base64urlEncode(signature)
```

[HMAC-SHA256 Online Generator Tool](https://www.devglan.com/online-tools/hmac-sha256-online)

온라인 툴로 해싱하면 해싱 값은
`6c0d16d7b7893d1a9e043d2137dc78e202ac716c5ca7f4842ad1bdcd237c0d1d`
이고 다시 base64로 인코딩하면
`NmMwZDE2ZDdiNzg5M2QxYTllMDQzZDIxMzdkYzc4ZTIwMmFjNzE2YzVjYTdmNDg0MmFkMWJkY2QyMzdjMGQxZA`
이다.

최종적으로 만들어진 JSON Web Token은

    eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJleGFtcGxlLmNvbSIsImh0dHBzOi8vZXhhbXBsZS5jb20vand0X2NsYWltcy9pc19hZG1pbiI6dHJ1ZSwidXNlcm5hbWUiOiJleHVzZXIxIn0.NmMwZDE2ZDdiNzg5M2QxYTllMDQzZDIxMzdkYzc4ZTIwMmFjNzE2YzVjYTdmNDg0MmFkMWJkY2QyMzdjMGQxZA

이다.

[JWT Decoder](https://www.jstoolset.com/jwt)
디코딩 해보면 정상적으로 디코딩되는 것을 확인할 수 있다.

![jwt-result](../../img/jwt-result.png)

## 주의 사항 및 단점

1. 자가 수용적: 토큰 자체에 정보를 담고 있으므로 보안의 취약점이 될 수 있다.
2. 토큰 길이: 토큰의 페이로드에 3종류의 클레임을 저장하기 때문에, 정보가 많아질수록 토큰의 길이가 늘어나 네트워크에 부하를 줄 수 있다.
3. 페이로드 인코딩: 페이로드 자체는 암호화 된 것이 아니라, BASE64로 인코딩 된 것이다. 중간에 Payload를 탈취하여 디코딩하면 데이터를 볼 수 있으므로, JWE로 암호화하거나 Payload에 중요 데이터를 넣지 않아야 한다.
4. 무상태성: JWT는 상태를 저장하지 않기 때문에 한번 만들어지면 제어가 불가능하다. 즉, 토큰을 임의로 삭제하는 것이 불가능하므로 토큰 만료 시간을 꼭 넣어주어야 한다.

# References

_https://bcho.tistory.com/999_
_https://velopert.com/2389_
_https://mangkyu.tistory.com/56_
