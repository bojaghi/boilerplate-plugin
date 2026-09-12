# 셋업 가이드

## 플러그인 셋업

셋업 가이드는 wp-env를 기준으로 설명합니다.
wp-env가 설치되어 있지
않다면, [wp-env 설치 가이드](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/)를 참고하십시오.

### 플러그인 위치

워드프레스의 루트 디렉토리에는 README.md, index.html, index.php, wp-config-sample.php 같은 파일이 있고,
wp-admin, wp-includes, wp-contents 같은 디랙토리가 있습니다.
본 플러그인 코드를 wp-content/plugins 디렉토리에 두면, 웓프레스 코어가 플러그인으로 인식합니다.

wp-env 환경에서는 필요한 기본적인 셋업이 되어 있으므로 플러그인을 반드시 코어의 정해진 디렉토리에 둘 필요는 없습니다.

### 테스트용 데이터베이스 계정 준비

wp-env를 사용하는 경우는 이 과정을 건너뛰어도 좋습니다.

워드프레스의 유닛 테스트를 위해서는 테스트 데이터베이스가 필요합니다.
테스트 데이터베이스를 생성할 수 있는 계정, 가령 MySQL 서버의 root 계정 같은 것이 필요합니다.
테스트를 진행하는 데이터베이스이므로 아무렇게나 쓰고 지워도 괜찮은 임시 데이터베이스를 사용하는 것이 좋습니다.

참고로 유닛 테스트르 위해 사용하는 기본값은 아래와 같습니다.

- 데이터베이스 이름: `wordpress_test`
- 데이터베이스 사용자: `wordpress_test`
- 데이터베이스 비밀번호: `wordpress_test`
- 데이터베이스 호스트: `localhost`

만약 커맨드라인 명령어를 통해 직접 계정을 생성하려면 아래를 참조하십시오.

```mysql
CREATE USER IF NOT EXISTS 'wordpress_test'@'localhost' IDENTIFIED BY 'wordpress_test';
GRANT ALL PRIVILEGES ON `wordpress_test`.* TO 'wordpress_test'@'localhost';
```

### WP-CLI 준비

유닛 테스트 준비 등을 위해, WP-CLI를 사용합니다. wp-env를 사용하는 경우, `npm run wp`로 WP-CLI를 실행할 수 있습니다.

### 워드프레스 동작 확인

wp-env를 사용하는 경우 `npm run wp-env:start` 명령으로 워드프레스를 실행할 수 있습니다.
`http://localhost:8888/wp-admin`, 사용자 이름 'admin', 비밀번호 'password'로 로그인할 수 있습니다.

### 유닛 테스트 준비 1: 스캐폴딩

WP-CLI를 이용해 플러그인의 유닛 테스트 준비를 진행합니다.

```shell
npm run wp -- scaffold plugin-tests boilerplate-plugin
```

스캐폴딩이 올바르게 진행되면 유닛 테스트를 위한 기본적인 준비가 완료됩니다. 다음 파일이 생성되었는지 확인하세요

- bin/install-wp-tests.sh
- tests/bootstrap.php
- tests/test-sample.php
- .phpcs.xml.dist
- phpunit.xml.dist

### 유닛 테스트 준비 2: PhpUnit 준비

composer.json에 이미 설정이 되어 있으므로, 아래 명령을 실행하면 됩니다.

```shell
npm run composer -- install
```

이제 `WP_TESTS_PHPUNIT_POLYFILLS_PATH` 상수의 경로가 올바르게 되어 있는지 확인합니다.
wp-env를 사용하면 이 상수의 값이 적절히 설정되어 있으나, 다른 환경일 경우 올바르게 맞춰 줄 필요가 있습니다.

이제 `phpunit.xml.dist`를 `phpunit.xml`로 이름을 변경, 또는 복사합니다.
만약 wp-env를 사용하지 않는 경우, 이 XML 파일에서 `WP_TESTS_PHPUNIT_POLYFILLS_PATH` 상수를 설정할 수도 있습니다.

```xml

<phpunit>
    ....
    <php>
        <constant name="WP_TESTS_PHPUNIT_POLYFILLS_PATH" value="vendor/yoast/phpunit-polyfills"/>
    </php>
</phpunit>
```

### 유닛 테스트 점검

`phpunit.xml` 내부에 `<exclude>./tests/test-sample.php</exclude>` 부분을 주석 처리합니다.
주석 처리는 아래처럼 합니다.

```
<!-- <exclude>./tests/test-sample.php</exclude> -->
```

주석 처리 후, `npm run phpunit`을 실행하여, 샘플 테스트가 올바로 진행되는지 확인합니다.
확인 후, 주석된 코드를 복원합니다.

### 레퍼런스 코드 설정

wp-env를 통해 워드프레스의 실행은 도커에서 진행되지만, 실제 코드를 작성할 때 코어의 소스나 워드프레스 테스트 수트 코드 소스가 없으면,
개발할 때 코드의 참조를 할 수가 없어 불편합니다. 이를 위해 tests 디렉토리에 코어와 테스트 수트 코드를 받습니다.

wp-env로 설치된 도커 환경에는 Subversion이 설치되어 있지 않습니다. 그러므로 이 명령은 로컬에서 Subversion을 설치 후 진행하는 것이 좋습니다.

```shell
# 플러그인의 루트 디렉토리에서,
svn --version --quiet # Subversion 버전 확인 
chmod +x ./bin/install-wp-tests.sh # 실행 권한을 줍니다.
rm -rf ./tests/wp-core ./tests/wp-tests # 우선 기존에 있을 수도 있는 디렉토리를 완전 삭제합니다.

# 코드 레퍼런스를 받습니다. 아래 두 줄을 모두 복사하여 붙여 넣습니다.
WP_CORE_DIR=./tests/wp-core WP_TESTS_DIR=./tests/wp-tests \
./bin/install-wp-tests.sh db user pass localhost 7.1 true
```

마지막 인자를 'true'로 하여 데이터베이스 생성을 생략하였기 때문에, 첫번째부터 네번째 인자는 크게 중요하지 않습니다.
적당히 채워 넣어도 무방합니다. 다만 '7.1'은 설치할 워드프레스 코어의 버전이기 때문에, 도커에 설치된 워드프레스 버전과 맞추는 것이 좋습니다.

이 셋업은 wp-env의 도커 내부의 설정과는 무관하며, 단지 코드 에디터에서 쿨래스, 함수 등의 심볼을 확인하고, xdebug에서 breakpoint를 위한
코드 레퍼런스로 사용하기 위해 진행합니다. 필요가 없다면 하지 않으셔도 됩니다.

### xdebug 원격 디버깅

wp-env 실행을 `npm run wp-env:start:xdebug`로 하면 원격 디버깅을 할 수 있습니다.

### 기타

- `.wp-env.json` 파일의 설정이 수정되었다면, wp-env를 멈추고 다시 실행해야 변경한 사항이 적용됩니다.
- `.gitignore` 파일은 실제 플러그인 개발 환경에 잘 맞지 않습니다. 적절히 수정하여 개발 환경에 맞게 조정하세요.
- 플러그인의 이름이 바뀌므로 반드시 모든 파일에 적힌 'boilerplate-plugin' 부분을 알맞게 수정해야 합니다.
  아래 파일 목록에서 모든 문자열이 올바르게 변경되었는지 확인하세요.
    -  .gitignore
    - .wp-env.json
    - boilerplate-plugin.php
    - composer.json
    - package.json
    - README.md
