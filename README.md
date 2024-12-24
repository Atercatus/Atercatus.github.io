# Atercatus.github.io

### 로컬 서버 실행

```
bundle exec jekyll serve
```

http://localhost:4000에서 웹사이트를 확인할 수 있습니다.
실시간으로 변경 사항이 반영됩니다.

### 정적 파일 생성 (빌드)

```
bundle exec jekyll build
```

\_site 디렉토리에 정적 파일이 생성됩니다.
이 파일들을 GitHub Pages나 다른 서버에 업로드하면 됩니다.

### 초안(Draft) 미리보기

\_drafts 디렉토리에 작성한 초안을 로컬 서버에서 확인하려면 다음 명령어를 사용하세요.

```
bundle exec jekyll serve --drafts
```

### 깨진 링크 확인

사이트에서 깨진 링크를 확인하려면 다음 명령어를 실행하세요.

```
jekyll doctor
```

### 코드 입력

```
{% highlight ruby %}
def print_hi(name)
puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}
```
