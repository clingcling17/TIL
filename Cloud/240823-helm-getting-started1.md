
# Helm

## Helm 템플릿 작성 가이드 
v3.14.0  
https://helm.sh/docs/chart_template_guide/getting_started/
  
### 기본 개념
- helm은 helm은 template 디렉토리 내부의 파일들을 기반으로 release의 manifest를 불러오고 설치한다.

- --dry-run 옵션을 통해서 릴리즈를 실제로 설치하지 않고 helm이 렌더링한 manifest 결과를 확인할 수 있다.

    - 예시) helm install geared-marsupi ./mychart --dry-run --debug

### Built-In Objects

- Template에 속한 Object는 다른 object와 function을 가진다.

- build-in Object의 종류: Release, Values, Chart, Subcharts, Files, Capabilities, Template

### Values

- template 렌더링에 쓰이는 values는 values.yaml, 부모 차트, 인라인 옵션 등에서 가져올 수 있다.

### Template Functions and Pipelines

- Go template language와 sprig 외 helm에서 제공하는 자체 function을 템플릿에 적용할 수 있다.

- Unix style pipeline을 템플릿에 적용할 수 있다.

- lookup function을 사용하여 k8s 클러스터 내의 리소스를 확인할 수 있다.

    - (단, –dry-run=server 옵션을 사용해야 테스트 가능)

### Template Function List

-   로직 관련
    - eq, ne, lt, gt, ge, and, or, default, required, empty, fail …
    
-   문자열 관련
    - print, println, printf, trim, trimAll, …
    
-   타입 변환
    - atoi, float64, int, int64, toDecimal, toString, toJson, fromYaml…
    
-   Regex 관련
    - regexMatch, mustRegexMatch, regexFindAll…
    
-   암호화 및 보안
    - sha1sum, derivePassword, genPrivateKey, buildCustomCert …
    
-   날짜 관련
    - now, ago, date, dateInZone, duration …
    
-   python-style 딕셔너리 관련
    - dict, get, set, …
    
-   리스트 관련
    - first, mustFirst, rest, last …
    
-   수학
    - add, add1, sub, div, mod, addf, add1f, subf …
    
-   네트워크 관련
    - getHostByName
    
-   경로 관련
    - base, dir, clean, ext, isAbs
    
-   타입 리플렉션 
    - kindOf, kindIs, typeOf, typeIs, deepEqual…
    
-   Semantic Versioning 관련
    - semver, semverCompare, 비교 연산자 등
    
-   URL 관련 
    - urlParse, urlJoin, urlquery
    
-   UUID 관련 
    - uuidv4
    
-   쿠버네티스 및 차트 관련
    - lookup, .Capabilities.APIVersions.Has 외 파일 관련
