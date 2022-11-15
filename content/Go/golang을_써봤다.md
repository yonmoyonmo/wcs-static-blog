---
title: "Golang을 써봤다."
date: 2022-11-15T19:58:50+09:00
draft: false
---

![gopher](/images/Go/gologo.png)

나의 쪼꼬만 앱 서비스 스몰토크헬퍼의 서버가 죽었다. 다시 살리려다가 go를 써볼 겸 다시 만들기로 했다. 만들면서 알게된 부분들을 기록해놓고 까먹었을 때 다시 보고 기억하기 위해 글을 쓰는 중이다.

[완성된 서버 소스코드](https://github.com/yonmoyonmo/new_small_talk_helper_server/tree/main/newServer)

새로운 언어를 익힐때 [공식문서](https://go.dev/doc/tutorial/getting-started)를 보고 하라는대로 해보는게 최고다. 나도 그렇게 한 번 따라해보고 Go의 컨셉을 어느정도 익혔다. 그 후 [freeCodeCamp go tutorial](https://www.youtube.com/watch?v=YS4e4q9oBaU)을 보고 조금 더 배웠다. 언어 그 자체에 대한 정보는 언제든지 다른 곳에서 찾아 볼 수 있으므로 적어놓지 않아도 될 것 같다.

이 글에는 HTTP로 API를 제공하는 서버를 만들기 위해 필요했던 golang 패키지들을 까먹지 않게 적어놓아야겠다.

## [net/http](https://pkg.go.dev/net/http)

Go는 기본 http 라이브러리로도 충분하다고 하길래 다른 프레임워크 없이 작업했다. 실제로 충분했다. Gin 등의 괜찮은 HTTP web 프레임워크가 많이 있으니 다음엔 써봐야지.

특정 포트로 서버를 대기시키고, 특정 url로 들어온 http 요청을 처리해서 적당히 응답해주는 동작을 할 수 있도록 API를 제공해준다. 몇 가지 type 등의 이름이 널리 사용되는 용어와 좀 달라서 뭔가 싶어도 자세히 읽어보면 대강 흔한 MVC 패턴 내로 다 녹여낼 수 있다.

예로 [ServeMux](https://pkg.go.dev/net/http#ServeMux)라는 타입이 있는데, 대략 router 기능을 하는구나 하고 이해하고 넘어가도 대충 쓸 수 있다. 이 때 사용되는 handler라는 용어도 MVC에서 controller의 역할을 할 수 있겠구나 싶어 그렇게 사용했다.

아래 코드처럼 route를 등록하여 ServeMux를 세팅하여 사용하니 좋았다.

```
func InitializeRouter() *http.ServeMux {
	log.Println("initializing router...")
	mux := http.NewServeMux()

	smthpBaseURL := "/api/sugguestion/small-talk-helper"
	mux.HandleFunc(smthpBaseURL+"/random", handler.RandomSuggHandler)
	mux.HandleFunc(smthpBaseURL+"/love36", handler.Love36Handler)
	mux.HandleFunc(smthpBaseURL+"/topten", handler.ToptenHandler)
	...생략...

	return mux
}

....메인.....
func main() {
	err := godotenv.Load()
	if err != nil {
		log.Fatal("Error loading .env file")
	}
	port := os.Getenv("PORT")
	log.Println("new small talk helper server")
	mux := router.InitializeRouter()
	log.Println("Listening...")
	log.Fatal(http.ListenAndServe(port, mux))
}
```

## [bcrypt](https://pkg.go.dev/golang.org/x/crypto/bcrypt)

비밀번호를 해싱해서 데이터베이스에 저장할 때 흔히 쓰이는 bcrypt 방식 역시 go package로 구현되어 있어서 그걸 사용했다.

## [jwt](https://pkg.go.dev/github.com/golang-jwt/jwt#section-readme)

request를 Authorizing하기 위해 JSON web token을 자주 사용하게 되는데, go에 역시 구현된 패키지가 있어서 그걸 사용했다. JWT에 익숙하다면 쉽게 사용할 수 있도록 잘 만들어져 있다.

## [database/sql](https://pkg.go.dev/database/sql) & [MySQL driver](https://pkg.go.dev/github.com/go-sql-driver/mysql)

MySQL 데이터베이스를 사용하기 위해 필요한 go 패키지들이다. sql 패키지가 일반적인 SQL 데이터베이스 인터페이스를 제공해준다. 이에 맞게 구현된 driver를 함께 쓰면 된다.

## 써 본 소감

Go는 참 매력적인 언어 같다. 재밌다. 심플한 문법이 높은 자유도를 준다. 너무 자유로워서 코드 정리나 디렉토리 정리를 어떻게 해야할지 모르겠다. 여기저기 뒤져보니 [비공식 스탠다드 레이아웃](https://github.com/golang-standards/project-layout) 같은게 있긴 하던데, 뭐가 됐든 컴파일만 잘 된다면 맘대루(개인작업물 한에서; 협업 시에는 상관 있다) 코드를 정리해도 상관없을지도? 나도 이번에 걍 내 맘대로 했다. 나중에 웹 말고 [다른 것(하단 what's possible with Go)](https://go.dev/)들도 Go로 만들어봐야지.

## 끝
