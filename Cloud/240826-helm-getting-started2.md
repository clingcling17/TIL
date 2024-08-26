
# Helm

## Helm 템플릿 작성 가이드 2
v3.14.0  
https://helm.sh/docs/chart_template_guide/control_structures/
  
### Flow Control
 - Helm 템플릿은 흐름 제어를 위하여 다음과 같은 제어문(action)을 제공한다.
	- if/else: 조건문
	- with: 스코프 지정
	- range: 반복문
	
	
- 선언을 위해서 제공되는 action은 다음과 같다.
	- define: 템플릿 내부에 named template 정의
	- template: named template을 불러옴
	- block 
	

#### If/Else

> {{ if PIPELINE }}  
>   \# Do something  
> {{ else if OTHER PIPELINE }}  
>   \# Do something else  
> {{ else }}  
>   \# Default case  
> {{ end }}

- pipeline은 다음과 같은 경우에 false 처리된다.
    - false, 0, "", nil, []

#### 공백 문자 처리
 
중괄호의 뒤에 - 를 붙여서 좌/우의 공백 문자를 제거할 수 있다. 
> {{- 3 }}

indent 함수를 사용하여 indentation을 조절할 수 있다.
> {{ indent 2 "mug:true" }}

#### with 액션

with 문을 사용하여 현재 스코프를 지정할 수 있다.
$를 사용하여 parent scope에 접근한다.

#### range 액션

- range 액션 내부에서는 slice의 객체가 스코프로 변화한다.
- yaml에서 |- 마커는 여러 줄로 된 문자열을 나타낸다.
> sizes: |-  
> &nbsp;&nbsp;{{- range tuple "small" "medium" "large" }}  
> &nbsp;&nbsp;&nbsp;&nbsp;- {{ . }}  
> &nbsp;&nbsp;{{- end }}      

- - -

### Variables

- Helm 템플릿에서 변수는 다른 오브젝트를 참조한다.

> {{- $relname := .Release.Name -}}  
> release: {{ $relname }}

- 정수 인덱스 사용
>  toppings: |-  
>  &nbsp;&nbsp;{{- range $index, $topping := .Values.pizzaToppings }}  
>  &nbsp;&nbsp;&nbsp;&nbsp;{{ $index }}: {{ $topping }}  
>  &nbsp;&nbsp;{{- end }}      

- key, value가 있는 자료구조
> {{- range $key, $val := .Values.favorite }}  
>  {{ $key }}: {{ $val | quote }}  
> {{- end }}  

- root context("$")를 제외하고, 변수 사용 스코프는 블록 내에 한한다.

- - -
### Named Templates

- named template은 파일 내에서 정의되고 이름을 가진다.
- 템플릿 이름은 전역으로 정의된다.
- 중복 방지를 위해 보통 차트 이름을 prefix로 넣는다.
  > {{ define "mychart.labels" }}
- templates/ 디렉토리 내부에서는 NOTES.txt와 _로 시작하는 파일을 제외하고 k8s manifest로 간주한다.

#### define과 template
- define 액션을 사용해서 네임드 템플릿을 생성할 수 있다.
> {{- define "MY.NAME" }}  
> &nbsp;&nbsp; \# body of template here  
> {{- end }}  

- 재사용할 부분은 기본적으로 _helpers.tpl 파일에 저장한다.

#### 템플릿의 범위 설정
named template을 사용할 때에는 template 문에 지정된 scope를 받는다.  
> {{- template "mychart.labels" **.** }}

#### include 
template 문의 결과는 문자열이므로, 현재 파이프라인에 내용을 불러오기 위해서는 include를 사용한다.
> {{ include "mychart.app" . | indent 2 }}

- - -

### 템플릿 안에서 파일 접근

- .Files 오브젝트를 이용하여 파일에 접근할 수 있다.
  - 차트는 1M 이하여야 하므로 파일 추가 시 주의한다.
  - templates/ 하위 파일, .helmignore에 설정된 파일, subchart 외부 파일은 접근할 수 없다.

- Go lang의 path 패키지 함수 중 일부를 사용할 수 있다.
  - Base, Dir, Ext, IsAbs, Clean

- Files.Glob(pattern string) 메소드를 사용할 수 있다.
  - Glob: 파일 이름의 집합을 wildcard characters로 명시하는 패턴  

  > {{ range \$path, \$_ :=  .Files.Glob  "**.yaml" }}  
  >      {{ $.Files.Get $path }}  
  > {{ end }}  

- 파일 내용을 ConfigMap 혹은 Secret으로 불러올 수 있다.
  > data:  
  > {{ (.Files.Glob "foo/*").AsConfig | indent 2 }}

- 안정성을 위하여 불러온 파일을 base64 인코딩할 수 있다.
  > data:  
  > &nbsp;&nbsp;token: |-  
  > &nbsp;&nbsp;&nbsp;&nbsp;{{ .Files.Get "config1.toml" | b64enc }}

- Files.Lines 함수를 사용하여 파일의 각 라인에 접근할 수 있다.

- - -

### NOTES.txt 파일 생성
templates/NOTES.txt 파일의 내용은 차트 install 시에 출력된다.

- - -

### SubChart and Global Values

- 차트는 subchart를 디펜던시로 가질 수 있다.
- subchart는 독립적이며, 부모의 값에 접근할 수 없다.
- 부모는 subchart의 값을 덮어쓸 수 있다.
- 모든 차트에서 접근 가능한 전역 값이 있다.
  > global:  
  > &nbsp;&nbsp; salad: caesar

- 부모 차트와 서브차트는 템플릿을 공유할 수 있다.
- block 문보다는 include 문을 사용하는 것이 좋다.

- - -

### .helmignore 파일
 - .helmignore 파일에 기재된 파일 목록은 helm package 명령어에서 무시한다.
 - 줄당 하나의 glob 매칭과 negation(!) 사용이 가능하다.

  > \# Match any file named ab.txt, ac.txt, or ad.txt  
  > a[b-d].txt  
  >   
  > \# Match any file under subdir matching temp*  
  > */temp 

- - -

### 템플릿 디버깅

템플릿 디버깅 시에 사용할 수 있는 명령어
 - helm lint
   - 차트가 best practice에 맞는지 체크
 - helm template --debug
 - helm install --dry-run --debug (--dry-run=server)
   - 위와 동작이 같으나 클러스터에 충돌하는 리소스가 있는지 확인
 - helm get manifest
 