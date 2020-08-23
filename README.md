### [탤로우 월드 블로그](https://taehyungk.github.io/)

- 꾸준히 기록하며 기억하게 하는 블로그를 만들자


#### 참고 사항
* 글 포스팅 후 Category, Tags, Archives 적용을 위해 터미널에서 `bash tools/publish.sh` 실행하여 오토 머지 필요함 !
  * jekyll **Chirpy** 테마
* Local에서 지킬 적용 사항 확인하기 위한 Bundler, Ruby 설치
```command
brew update
brew install rbenv ruby-build
```
  * rbenv 설치 확인
  ```command
  rbenv versions
  ```
  * 설치 가능한 Ruby 버전 조회
  ```command
  rbenv install -l
  ```
  * 버전 설치
  ```command
  rbenv install 2.7.1
  ```
  * 글로벌로 설정
  ```command
  rbenv global 2.7.1
  ```
  * rbenv PATH를 추가를 위해 쉘 진입
  ```command
  vim ~/.zshrc  
  ```
  * rbenv PATH 추가
  ```command
  export PATH=${HOME}/.rbenv/bin:${PATH} && \
  eval "$(rbenv init -)"
  ```
  * PATH 적용
  ```command
  source ~/.zshrc
  ```
  * Bundler 설치
  ```command
  gem install bundler
  ```
  
