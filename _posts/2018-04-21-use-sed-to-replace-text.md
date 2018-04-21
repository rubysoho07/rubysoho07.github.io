---
layout: post
title: "sed를 이용한 텍스트 파일 바꾸기"
categories: Linux
tags: Linux, sed
comments: false
---

쉘 스크립트를 만들 때, 텍스트 파일의 내용을 바꾸는 것이 필요할 때가 있다. 이 경우 `sed`를 쓰는데, sed의 사용 방법이 헷갈릴 때가 많아 이 기회에 정리를 해 보고자 한다. 이 글에서는 기본적인 사용 방법과 내가 주로 사용하는 기능만 정리하였다. sed가 지원하는 전체 기능이 궁금하다면, man page(`man sed`)를 확인하는 것이 최선이다. 

### 텍스트 바꾸기

기본적으로 sed를 이용해서 텍스트를 바꾸는 방법은 다음과 같다.

```
$ sed -i 's/<원래 내용>/<바꿀 내용>/g' <파일 이름>
```

여기서 `-i`, `s/<원래 내용>/<바꿀 내용>/`, `g`가 의미하는 내용은 다음과 같다.

* `-i`: 변경할 내용을 그 파일에 바로 반영해서 저장한다. 
* `s/<원래 내용>/<바꿀 내용>/`: `원래 내용`에 적은 것을 `바꿀 내용`으로 치환한다. 일반 텍스트 및 정규표현식을 쓸 수 있다. 
* `g`: 한 행에 같은 내용이 여러 번 나타나면, 모두 바꾼다. (없으면 그 행의 맨 처음으로 나오는 것만 바꾼다.)

### 정규표현식을 이용한 텍스트 바꾸기

위에서도 이야기 했듯 sed는 일반 텍스트 및 정규표현식을 이용하여 텍스트를 바꿀 수 있다. 정규표현식에 익숙하지 않다면 [RegExr](https://regexr.com/)에서 연습해 보기를 추천한다.

#### 예제 1: 패턴을 사용해서 텍스트 바꾸기

이런 파일이 있고, npm와 nvm을 모두 aaa로 바꾸고 싶다고 가정하자. (파일 이름은 `test.txt`로 저장했다)

```
 1021  nvm list
 1022  nvm use stable
 1023  npm list
 1024  npm list --global
```

nvm이나 npm의 차이는 중간에 있는 `v`와 `p`이다. 중간에 있는 한 글자 차이는 `.`을 이용해서 바꾸면 되므로 다음과 같이 실행하면 된다. 

```
$ sed -i 's/n.m/aaa/g' test.txt
```

결과는 다음과 같다.

```
$ cat test.txt
 1021  aaa list
 1022  aaa use stable
 1023  aaa list
 1024  aaa list --global
```

### 예제 2: 패턴 안에 `/`와 같은 특수문자가 들어가는 경우

그런데, 내가 대체하려는 패턴 안에 `/`와 같은 특수문자가 들어가면 어떻게 해야 할까? 이 경우 `\`를 앞에 붙여주면 된다. 

`test.txt` 파일에서 `com/example`을 `net/example`로 바꾸려고 한다. 

원본 파일은 다음과 같다. 

```
$ cat test.txt 
 1084  git clone https://github.com/example/caliper-data-example
 1085  git clone https://github.com/example/caliper-data-test.git
 1126  git clone https://github.com/example/caliper-dashboard-test.git
 1133  git remote add upstream  https://gitlab.com/example/caliper-python-example.git
 1138  git clone https://gitlab.com/example/caliper-python-example.git
 1139  git clone https://gitlab.com/example/caliper-dashboard-test.git
```

다음과 같이 `/` 앞에 `\`를 붙여서 바꿔준다.

```
$ sed -i 's/com\/example/net\/example/g' test.txt
```

결과는 다음과 같다. 

```
$ cat test.txt
 1084  git clone https://github.net/example/caliper-data-example
 1085  git clone https://github.net/example/caliper-data-test.git
 1126  git clone https://github.net/example/caliper-dashboard-test.git
 1133  git remote add upstream  https://gitlab.net/example/caliper-python-example.git
 1138  git clone https://gitlab.net/example/caliper-python-example.git
 1139  git clone https://gitlab.net/example/caliper-dashboard-test.git
```

#### 예제 3: 검색 결과의 일부를 대체할 내용에서 사용하고 싶을 때

패턴으로 검색한 결과의 일부를 대체할 내용에도 쓰고 싶을 때가 있을 것이다. 이 경우 그룹으로 만들 부분을 괄호로 묶어준다. (`\(<패턴>\)`과 같이 묶어주면 된다. ) 그리고 묶은 그룹은 `\1`과 같이 순서대로 쓰면 된다. (man page에서는 일치하는 하위 패턴을 가리킬 때 `\1`에서부터 `\9`까지 쓸 수 있다고 나와 있다.)

다음과 같은 텍스트 파일에서 `pip`을 `python -m pip`으로 바꾸려고 한다. (더 편한 방법도 있겠지만, 설명을 위해 다음 예제를 사용한다.)

```
$ cat test.txt
 1253  pip install selenium
 1254  pip freeze | grep selenium
 1255  pip install pytz
```

`pip` 부분을 `\(pip\)`으로 묶고, 대체할 내용에 `\1`을 넣어 찾은 패턴의 결과가 들어가도록 만들어 준다. 
아래 내용을 실행한다.

```
$ sed -i 's/\(pip\)/python -m \1/g' test.txt
```

결과는 다음과 같다.

```
$ cat test.txt
 1253  python -m pip install selenium
 1254  python -m pip freeze | grep selenium
 1255  python -m pip install pytz
```

### 여담: macOS에서 `invalid command code ., despite escaping periods` 오류가 발생하는 경우

예전에 macOS에서 sed를 썼을 때, 위와 같은 명령을 썼음에도 불구하고 `invalid command code ., despite escaping periods` 오류가 발생한 적이 있다. 

이 경우 다음과 같이 `-i` 뒤에 `''`를 넣고, 패턴 앞에 `-e` 옵션을 넣어 해결하면 된다. 

원래 명령: `sed -i 's/n.m/aaa/g' test.txt`
macOS에서 쓰는 방법: `sed -i '' -e 's/n.m/aaa/g' test.txt`

#### 참고한 내용들

[https://stackoverflow.com/questions/19456518/invalid-command-code-despite-escaping-periods-using-sed](https://stackoverflow.com/questions/19456518/invalid-command-code-despite-escaping-periods-using-sed)
