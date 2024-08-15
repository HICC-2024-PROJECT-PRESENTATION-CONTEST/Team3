# Hi, CC! API 명세서

## 요청과 응답

API 요청과 응답에서 페이로드가 필요한 경우 기본적으로 JSON 형태의 페이로드를 사용합니다. 사진 데이터와 같은 경우에만 예외적으로 `multipart/form-data` 형태의 페이로드를 사용합니다.

접근 권한 인증은 클라이언트에 저장된 쿠키를 통해 세션 인증을 구현하여 사용합니다.

모든 프로필에는 [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) 형태의 고유 아이디가 부여됩니다. 아래에서 설명하는 것처럼 `:uid` 부분에 프로필의 고유 아이디를 넣어 요청 URL 을 만들어 사용하면 됩니다.

API URL 의 path 에서 `:val` 과 같은 형태의 표기는 변수를 의미합니다. 예를 들어 [프로필 정보 가져오기](#get-profilesuid-프로필-정보-가져오기) 를 실제 사용할 때의 URL 은 `/profiles/00000000-0000-0000-0000-000000000000` 형태로 만들어 사용하면 됩니다.

API 응답은 JSON 형태로 전송됩니다.
응답의 형태는 다음과 같습니다.

```json
{
  "message": "응답 메시지",
  "data": {}
}
```

- `message` 필드에는 응답 코드에 대한 추가적인 설명이 들어갑니다.
- `data` 필드에는 응답에 대한 바디 페이로드가 들어갑니다.

## API

- `/auth/qr`
  - [`POST` QR 생성](#post-authqr-qr-생성)
  - `/auth/qr/:key?redir`
    - [`GET` QR 사용 (사용자 로그인)](#get-authqrkeyredir-qr-사용-사용자-로그인)
- `/auth/login?redir`
  - [`POST` 사용자 로그인](#post-authloginredir-사용자-로그인)
- `/auth/manager/login/:key?redir`
  - [`POST` 관리자 로그인](#post-authloginredir-사용자-로그인)
- `/profiles`
  - [`GET` 프로필 목록 가져오기](#get-profiles-프로필-목록-가져오기)
  - [`POST` 새 프로필 생성](#post-profiles-새-프로필-생성)
  - `/profiles/:uid`
    - [`GET` 프로필 정보 가져오기](#get-profilesuid-프로필-정보-가져오기)
    - [`PATCH` 프로필 정보 수정](#patch-profilesuid-프로필-정보-수정)
    - [`DELETE` 프로필 정보 삭제](#delete-profilesuid-프로필-정보-삭제)
    - `/profiles/:uid/image`
      - [`GET` 프로필 사진 가져오기](#get-profilesuidimage-프로필-사진-가져오기)
      - [`PATCH` 프로필 사진 수정 (업로드)](#patch-profilesuidimage-프로필-사진-수정-업로드)
      - [`DELETE` 프로필 사진 삭제](#delete-profilesuidimage-프로필-사진-삭제)
    - `/profiles/:uid/recommands`
      - [`GET` 프로필의 추천 상대 목록 가져오기](#get-profilesuidrecommands-프로필의-추천-상대-목록-가져오기)
    - `/profiles/:uid/choices`
      - [`GET` 프로필이 / 프로필을 선택한 상대 목록 가져오기](#get-profilesuidchoices-프로필이--프로필을-선택한-상대-목록-가져오기)
      - [`POST` 프로필의 상대 선택](#post-profilesuidchoices-프로필의-상대-선택)
    - `/profiles/:uid/message`
      - [`POST` 프로필의 상대에게 메시지 보내기](#post-profilesuidmessage-프로필의-상대에게-메시지-보내기)

<br><br><br>

### `POST` `/auth/qr` QR 생성

`접근 권한: 관리자`

- 요청

  - 페이로드 없음

- 응답

  - `200` 성공

    - 바디 페이로드 (JSON)

      ```json
      {
        "key": "000000"
      }
      ```

  - `403` 접근 권한 없음
  - `500` 서버 오류

<br><br><br>

### `GET` `/auth/qr/:key?redir` QR 사용 (사용자 로그인)

`접근 권한: 없음`

- 요청

  - 쿼리 페이로드

    redir: 리다이렉트할 URL

- 응답

  - `302` 리다이렉트
  - `403` 접근 권한 없음
  - `500` 서버 오류

<br><br><br>

### `POST` `/auth/login?redir` 사용자 로그인

`접근 권한: 없음`

- 요청

  - 쿼리 페이로드

    redir: 리다이렉트할 URL

  - 바디 페이로드 (JSON)
    ```json
    {
      "phone": "01000000000",
      "password": "password"
    }
    ```

- 응답

  - `200` 성공
  - `400` 필수 필드 누락
  - `400` 잘못된 필드 형식
  - `401` 접근 권한 없음 (로그인 실패)
  - `500` 서버 오류

<br><br><br>

### `POST` `/auth/manager/login` 관리자 로그인

`접근 권한: 없음`

- 요청

  - 바디 페이로드 (JSON)

    ```json
    {
      "key": "000000"
    }
    ```

- 응답

  - `200` 성공
  - `400` 필수 필드 누락
  - `400` 잘못된 필드 형식
  - `401` 접근 권한 없음 (로그인 실패)
  - `500` 서버 오류

<br><br><br>

### `GET` `/profiles` 프로필 목록 가져오기

`접근 권한: 관리자`

- 요청

  - 쿼리 페이로드

    - `page` 페이지 (number)
    - `size` 페이지 당 개수 (number)
    - `find` 찾을 검색어 (string)
    - `count` 개수 (boolean)

    쿼리 페이로드를 통해 프로필 목록을 원하는 만큼만 가져올 수 있습니다. `find` 쿼리는 프로필의 이름, 전화번호, 인스타그램 아이디 안에서 `find` 태그의 내용을 검색합니다.
    `count` 쿼리를 사용하는 경우 `page`와 `size` 값은 무시되고 `find` 쿼리만 적용하여 개수를 셉니다.

    쿼리 예시

    - 대충 힉익명을 찾을 경우

      `/profiles?find=힉익명&page=1&size=20`

    - 전체 프로필 개수를 찾는 경우

      `/profiles?count`

- 응답

  - `200` 성공

    - 바디 페이로드 (JSON) (`count` 를 사용하지 않은 경우)

      ```json
      [
        {
          "uid": "00000000-0000-0000-0000-000000000000",
          "name": "히이익",
          "phone": "01000000000",
          "instagram": "hiiiiiccccc",
          "gender": "F",
          "birthyear": 2003,
          "birthyear_offset": {
            "plus": 3,
            "minus": 1
          },
          "height": 165,
          "major": "자율전공",
          "mbti": "ICBM",
          "looklike": "MISSILE",
          "smoking": false
        }
      ]
      ```

    - 바디 페이로드 (JSON) (`count` 를 사용하는 경우)

      ```json
      {
        "count": 200
      }
      ```

  - `403` 접근 권한 없음
  - `500` 서버 오류

<br><br><br>

### `POST` `/profiles` 새 프로필 생성

`접근 권한: 사용자`

- 요청

  - 바디 페이로드 (JSON)

    ```json
    {
      "name": "히이익",
      "phone": "01000000000",
      "instagram": "hiiiiiccccc",
      "password": "password",
      "gender": "M",
      "birthyear": 2003,
      "birthyear_offset": {
        "plus": 3,
        "minus": 1
      },
      "height": 175,
      "major": "자율전공",
      "mbti": "ICBM",
      "looklike": "MISSILE",
      "smoking": false
    }
    ```

  ```

  ```

- 응답

  - `200` 성공
  - `400` 필수 필드 누락
  - `400` 잘못된 필드 형식
  - `403` 접근 권한 없음
  - `409` 중복되는 이름과 전화번호
  - `500` 서버 오류

<br><br><br>

### `GET` `/profiles/:uid` 프로필 정보 가져오기

`접근 권한: 사용자 (본인), 관리자 (타인)`

- 요청

  - 페이로드 없음

- 응답

  - `200` 성공

    - 바디 페이로드 (JSON)
      ```json
      {
        "uid": "00000000-0000-0000-0000-000000000000",
        "name": "히이익",
        "phone": "01000000000",
        "instagram": "hiiiiiccccc",
        "gender": "M",
        "birthyear": 2003,
        "birthyear_offset": {
          "plus": 3,
          "minus": 1
        },
        "height": 175,
        "major": "자율전공",
        "mbti": "ICBM",
        "looklike": "MISSILE",
        "smoking": false,
        "choicescount": 1,
        "messagescount": 1
      }
      ```

  - `403` 접근 권한 없음
  - `404` 프로필 정보를 찾을 수 없음
  - `500` 서버 오류

<br><br><br>

### `PATCH` `/profiles/:uid` 프로필 정보 수정

`접근 권한: 사용자 (본인), 관리자 (타인)`

- 요청

  - 바디 페이로드 (JSON)

    ```json
    {
      "name": "히이익",
      "instagram": "hiiiiiccccc",
      "height": 175,
      "major": "자율전공",
      "mbti": "ICBM",
      "looklike": "MISSILE",
      "smoking": false
    }
    ```

- 응답

  - `200` 성공
  - `400` 필수 필드 누락
  - `400` 잘못된 필드 형식
  - `403` 접근 권한 없음
  - `404` 프로필 정보를 찾을 수 없음
  - `409` 중복되는 이름과 전화번호
  - `500` 서버 오류

<br><br><br>

### `DELETE` `/profiles/:uid` 프로필 정보 삭제

`접근 권한: 사용자 (본인), 관리자 (타인)`

- 요청

  - 페이로드 없음

- 응답

  - `200` 성공
  - `403` 접근 권한 없음
  - `404` 프로필 정보를 찾을 수 없음
  - `500` 서버 오류

<br><br><br>

### `GET` `/profiles/:uid/image` 프로필 사진 가져오기

`접근 권한: 없음`

- 요청

  - 쿼리 페이로드

    - `size` 사진의 크기를 지정합니다. 200, 900, 1500 을 지정할 수 있습니다. 지정하지 않으면 기본값은 900입니다.

- 응답

  - `200` 성공

    - 바디 페이로드 (이미지)

  - `403` 접근 권한 없음
  - `404` 프로필 정보를 찾을 수 없음
  - `500` 서버 오류

<br><br><br>

### `PATCH` `/profiles/:uid/image` 프로필 사진 수정 (업로드)

`접근 권한: 사용자 (본인), 관리자 (타인)`

- 요청

  - 바디 페이로드 (multipart/form-data)

    ```
    image=steam:data
    ```

    사용 가능한 이미지 mime 은 `image/jpeg`, `image/png`, `image/webp` 입니다. 서버에 저장될 때는 전부 리사이즈된 jpg 파일로 저장됩니다.

    업로드 예시 코드가 백엔드 리포지토리 `src/publix/examples/image-upload.html` 에 있습니다.

- 응답

  - `200` 성공
  - `400` 필수 필드 누락
  - `403` 접근 권한 없음
  - `404` 프로필 정보를 찾을 수 없음
  - `500` 서버 오류

<br><br><br>

### `DELETE` `/profiles/:uid/image` 프로필 사진 삭제

`접근 권한: 사용자 (본인), 관리자 (타인)`

- 요청

  - 페이로드 없음

- 응답

  - `200` 성공
  - `403` 접근 권한 없음
  - `404` 프로필 정보를 찾을 수 없음
  - `500` 서버 오류

<br><br><br>

### `GET` `/profiles/:uid/recommands` 프로필의 추천 상대 목록 가져오기

`접근 권한: 사용자 (본인), 관리자 (타인)`

- 요청

  - 페이로드 없음

- 응답

  - `200` 성공

    - 바디 페이로드 (JSON)

      ```json
      [
        {
          "uid": "00000000-0000-0000-0000-000000000000",
          "name": "히이익",
          "instagram": "hiiiiiccccc",
          "gender": "F",
          "birthyear": 2003,
          "height": 165,
          "major": "자율전공",
          "mbti": "ICBM",
          "looklike": "MISSILE",
          "smoking": false
        }
      ]
      ```

  - `403` 접근 권한 없음
  - `404` 프로필 정보를 찾을 수 없음
  - `500` 서버 오류

<br><br><br>

### `GET` `/profiles/:uid/choices` 프로필이 / 프로필을 선택한 상대 목록 가져오기

`접근 권한: 사용자 (본인), 관리자 (타인)`

- 요청

  - 페이로드 없음

- 응답

  - `200` 성공

    - 바디 페이로드 (JSON)

      ```json
      {
        "choices_to": [
          {
            "uid": "00000000-0000-0000-0000-000000000000",
            "name": "히이익",
            "instagram": "hiiiiiccccc",
            "gender": "F",
            "birthyear": 2003,
            "height": 165,
            "major": "자율전공",
            "mbti": "ICBM",
            "looklike": "MISSILE",
            "smoking": false
          },
          {
            "uid": "00000000-0000-0000-0000-000000000001",
            "delete": true
          }
        ],
        "choices_from": [
          {
            "uid": "00000000-0000-0000-0000-000000000002",
            "name": "히이익",
            "instagram": "hiiiiiccccc",
            "gender": "F",
            "birthyear": 2003,
            "height": 158,
            "major": "자율전공",
            "mbti": "ICBM",
            "looklike": "MISSILE",
            "smoking": true
          },
          {
            "uid": "00000000-0000-0000-0000-000000000003",
            "delete": true
          }
        ]
      }
      ```

  - `403` 접근 권한 없음
  - `404` 프로필 정보를 찾을 수 없음
  - `500` 서버 오류

<br><br><br>

### `POST` `/profiles/:uid/choices` 프로필의 상대 선택

`접근 권한: 사용자 (본인), 관리자 (타인)`

- 요청

  - 바디 페이로드 (JSON)

    ```json
    {
      "target": "00000000-0000-0000-0000-000000000000"
    }
    ```

- 응답

  - `200` 성공
  - `400` 필수 필드 누락
  - `400` 잘못된 필드 형식
  - `403` 접근 권한 없음
  - `404` 프로필 정보를 찾을 수 없음
  - `409` 상대 선택 횟수 없음
  - `500` 서버 오류

<br><br><br>

### `POST` `/profiles/:uid/message` 프로필의 상대에게 메시지 보내기

`접근 권한: 사용자 (본인), 관리자 (타인)`

- 요청

  - 바디 페이로드 (JSON)

    ```json
    {
      "target": "00000000-0000-0000-0000-000000000000",
      "message": "메시지 내용"
    }
    ```

- 응답

  - `200` 성공
  - `400` 필수 필드 누락
  - `400` 잘못된 필드 형식
  - `403` 접근 권한 없음
  - `404` 프로필 정보를 찾을 수 없음
  - `409` 메시지 전송 횟수 없음
  - `500` 서버 오류
