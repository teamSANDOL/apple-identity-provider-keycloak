## 팀 fork 안내 (teamSANDOL)

이 저장소는 teamSANDOL이 [klausbetz/apple-identity-provider-keycloak](https://github.com/klausbetz/apple-identity-provider-keycloak)을 fork해서 수정한 버전입니다. 원본은 Klaus Betz와 기여자들이 만들었고, Apache License 2.0으로 배포됩니다.

- 원본 저장소: https://github.com/klausbetz/apple-identity-provider-keycloak
- 기준 버전: 상류 1.17.0 태그 (커밋 665011f)
- 이 fork의 버전: 1.17.0-sandol.1
- 대상 Keycloak: 26.8.1
- 작업 브랜치: `sandol/keycloak-26.8` (`main`은 상류와 맞추는 용도로만 씁니다)

이 브랜치에서 바꾼 내용

1. Keycloak 26.8.1 대응: `keycloakVersion`을 26.5.0에서 26.8.1로, Gradle wrapper를 8.0.2에서 8.14.6으로 올렸습니다. 26.8.1에서 메서드 이름이 바뀌어 `exchangeExternalTokenV1Impl`을 `exchangeExternalImpl`로 고쳤습니다.
2. ID token에 이메일이 없으면 Apple이 보낸 user JSON에서 이메일을 가져옵니다. 상류 main에 들어갔지만 아직 릴리스되지 않은 커밋 7e087bc를 cherry-pick했습니다.
3. Apple 토큰 응답이 없으면(`NoHttpResponseException`) 토큰 요청을 한 번 다시 보냅니다([상류 #116](https://github.com/klausbetz/apple-identity-provider-keycloak/issues/116)). 인가 코드가 만료되는 등의 이유로 토큰 교환에 실패하면 502 오류 대신 로그인 화면으로 돌려보냅니다([상류 #118](https://github.com/klausbetz/apple-identity-provider-keycloak/issues/118)). 상류 `114-fix` 브랜치에서 아직 병합되지 않은 커밋 085a73a를 cherry-pick했습니다.
4. `.gitignore`에 `.vscode/`와 `bin/`을 추가했습니다.

바꾼 파일과 이유는 [NOTICE](NOTICE)에 정리했습니다. 아래부터는 원본 README입니다. 설치 안내에 나오는 JAR 링크와 호환표는 원본 저장소 기준입니다. 이 fork의 JAR은 [이 저장소의 Releases](https://github.com/teamSANDOL/apple-identity-provider-keycloak/releases)에서 받습니다.

---

# Apple Identity Provider for Keycloak :apple:

This repository represents an extension for [Keycloak](https://www.keycloak.org), which
enables [Sign in with Apple](https://developer.apple.com/documentation/sign_in_with_apple) for web-based and native
applications (via `token-exchange`).

### Apple vs. the rest of the world

Since Apple does not comply 100% to the existing `OpenID Connect` standard, some customizations are necessary in order to make the _Apple
way_
compatible to Keycloak. Differences are as follows:

- If scopes were requested, Apple sends the `TokenResponse` as a `POST` request
- There's no `userinfo` endpoint. `email`, `firstName` and `lastName` are transmitted only the first time the user signs in with Apple.
- The `/token`-request must contain a client-secret (JWT) which is signed using a specific private key (`.p8` file).

## Installation

1. Make sure you have a copy of the latest [JAR](https://github.com/klausbetz/apple-identity-provider-keycloak/releases/latest) or include
   it as a Maven dependency <a href="https://jitpack.io/#klausbetz/apple-identity-provider-keycloak">
   <img src="https://jitpack.io/v/klausbetz/apple-identity-provider-keycloak.svg" alt="Release"></a>.
2. Deploy the JAR file to keycloak (by placing it in `/opt/keycloak/providers` inside your docker container; alternatively see
   this [guide](https://www.keycloak.org/docs/latest/server_development/index.html#registering-provider-implementations))
3. Keycloak might need a restart (or a fresh container when using docker)

See a [`Dockerfile` example](docs/README_docker_installation.md).

## Upgrading your existing installation

Please refer to [UPGRADE.md](UPGRADE.md). In rare cases, you are required to perform some steps to make the extension work again after an
upgrade.

## Compatibility

Since this package implements internal Keycloak SPIs, there's a chance this extension breaks with a new major version of Keycloak.  
See the compatibility list below to find the version that suits your Keycloak version.

| Keycloak Version                        | Extension Version                 |
|-----------------------------------------|-----------------------------------|
| `< 17.0.0`                              | Not tested. Use at your own risk. |
| `17.0.0 <= 19.0.3`                      | `1.2.0`                           |
| `20.0.0 <= 20.0.5` :information_source: | `1.3.0 <= 1.4.1`                  |
| `21.0.0 <= 21.0.2` :information_source: | `1.5.0`                           |
| `21.1.0 <= 21.1.2` :information_source: | `1.6.0`                           |
| `22.0.0 < 23.0.0`                       | `1.7.0 <= 1.8.0`                  |
| `23.0.0 < 25.0.0`                       | `1.9.0 <= 1.12.0`                 |
| `25.0.0 < 26.2.3`                       | `1.13.0 <= 1.14.0`                |
| `26.3.0 < 26.4.7`                       | `1.15.0 <= 1.16.0`                |
| `>= 26.5.0`                             | `>= 1.17.0`                       |

:information_source: In Keycloak `v21.X.Y` this extension cannot be used effectively, since the additional properties such
as `Team ID`, `Key ID`
and so on are not displayed in the Admin UI.   
However, you can still use this extension with Keycloak `v21.X.Y` when you use some sort of external configuration tools like terraform or
[keycloak-config-cli](https://github.com/adorsys/keycloak-config-cli) where you do not rely on Keycloak's UI.

:information_source: For Keycloak `v19` and `v20` you have to switch to the old Admin UI to use this extension (see this
paper [Keycloak 19.0.0 release](https://www.keycloak.org/2022/07/keycloak-1900-released.html#_new_admin_console_is_now_the_default_console))

## Configuration

Log into your Keycloak admin console and add `Apple` as new `Identity Provider` and get comfortable with the configuration options:

| Option                                  | Description                                                                                                                                                                                                    |
|-----------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Client ID (= Service ID)                | Your Service ID from your Apple Developer Account.                                                                                                                                                             |
| Client Secret                           | Raw content of p8 key file you get from your Apple Developer Account.                                                                                                                                          |
| Team ID                                 | Your Team ID obtained from your Apple Developer Account.                                                                                                                                                       |
| Key ID                                  | A key identifier obtained from your Apple Developer Account.                                                                                                                                                   |
| p8 Key                                  | (replaced by Client Secret since `v1.14.0`) Raw content of p8 key file you get from your Apple Developer Account.                                                                                              |
| Default Scopes                          | Scopes to request from Apple (for web-based logins). Defaults to `name%20email`                                                                                                                                |
| Token-Exchange links existing accounts  | Since `1.16.0`. If enabled, accounts are automatically linked to existing keycloak accounts (since the flow `first broker login` is not invoked for token exchanges that would take care of that automatically |

:raising_hand: If you are unsure where you get these values from your Apple Developer Account, you may want to check out
this [guide](https://github.com/klausbetz/apple-identity-provider-keycloak/wiki/Configuration-within-Apple-Developer-portal).

:warning: Make sure to add the keycloak broker-URL (`https://<keycloak-url>/realms/<realm>/broker/apple/endpoint`) to your valid redirect
URLs in your Apple Developer Account.

## E-Mail configuration for private relay
In case a user registers using a hidden email address from Apple (private relay) the sending mail address of your Keycloak realm needs to be whitelisted in the Apple Developer account. Otherwise users with private relay addresses will not receive any emails from Keycloak.
[See this documentation](https://developer.apple.com/help/account/capabilities/configure-private-email-relay-service/)

## Troubleshooting

Please refer to the [wiki](https://github.com/klausbetz/apple-identity-provider-keycloak/wiki/Troubleshooting-the-configuration-from-Apple)
to figure out what's wrong.

## Token exchange

:point_right: If you are unsure whether you need token-exchange or not, check out
the [wiki](https://github.com/klausbetz/apple-identity-provider-keycloak/wiki/Do-you-need-token%E2%80%90exchange%3F).

Token exchange can be used to trade Apple tokens for Keycloak access- and refresh-tokens.  
This flow is mostly interesting for native applications like iOS apps, to provide native login options.

:point_right: For enabling token-exchange in Keycloak there is some configuration necessary (
see [Configure token-exchange](docs/CONFIGURE_TOKEN_EXCHANGE.md))

### 1. Using Apple ID-Token (recommended)

You want to use the `/token` endpoint of your realm to exchange an Apple ID-Token for Keycloak tokens.  
`<keycloak server url>/realms/<realm>/protocol/openid-connect/token`  
`application/x-www-form-urlencoded`

| Parameter            | Description                                                                                                                                                                                                       |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `client_id`          | the client id of your Keycloak client                                                                                                                                                                             |
| `grant_type`         | `urn:ietf:params:oauth:grant-type:token-exchange`                                                                                                                                                                 |
| `subject_token`      | ID-Token from Apple                                                                                                                                                                                               |
| `subject_issuer`     | `apple` (the name of the social provider in keycloak)                                                                                                                                                             |
| `subject_token_type` | `urn:ietf:params:oauth:token-type:id_token`                                                                                                                                                                       |
| `user_profile`       | `{ "name": { "firstName": string, "lastName": string }, "email": string }` optional. The JSON string that Apple sends on the first login (only required for the first login if you want to store the user's name) |

### 2. Using Apple `authorization_code`

![token exchange](docs/token_exchange.png)

#### Details

Looking at `3` from the image above, you want to use the `/token` endpoint of your realm to exchange an Apple `authorizationCode` for
Keycloak tokens.  
`<keycloak server url>/realms/<realm>/protocol/openid-connect/token`  
`application/x-www-form-urlencoded`

| Parameter          | Description                                                                                                                                                                                                        |
|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `client_id`        | the client id of your Keycloak client                                                                                                                                                                              |
| `grant_type`       | `urn:ietf:params:oauth:grant-type:token-exchange`                                                                                                                                                                  |
| `subject_token`    | `authorizationCode` from Apple                                                                                                                                                                                     |
| `subject_issuer`   | `apple` (the name of the social provider in keycloak)                                                                                                                                                              |
| `user_profile`     | `{ "name": { "firstName": string, "lastName": string }, "email": string }` optional. The JSON string that Apple sends on the first login (only required for the first login  if you want to store the user's name) |
| `app_identifier`   | the ServiceID/BundleID the authorization was invoked with. You can omit this field if the ServiceID in Keycloak is the same as the BundleID                                                                        |
| `app_redirect_uri` | the Redirect URI the authorization was invoked with (since `1.16.0`)                                                                                                                                               |

