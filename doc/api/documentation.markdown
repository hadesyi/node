# About this Documentation

<!-- type=misc -->

The goal of this documentation is to comprehensively explain the Node.js
API, both from a reference as well as a conceptual point of view.  Each
section describes a built-in module or high-level concept.

Where appropriate, property types, method arguments, and the arguments
provided to event handlers are detailed in a list underneath the topic
heading.

Every `.html` document has a corresponding `.json` document presenting
the same information in a structured manner.  This feature is
experimental, and added for the benefit of IDEs and other utilities that
wish to do programmatic things with the documentation.

Every `.html` and `.json` file is generated based on the corresponding
`.markdown` file in the `doc/api/` folder in node's source tree.  The
documentation is generated using the `tools/doc/generate.js` program.
The HTML template is located at `doc/template.html`.

<!-- type=misc -->

이 문서의 목적은 Node.js API를 레퍼런스부터 개념적인 부분까지 광범위하게 설명하는 
데 있다. 각 섹션은 내장(built-in) 모듈이나 고수준의 개념을 설명한다.

프로퍼티 타입, 메서드 아규먼트, 이벤트 핸들러의 아규먼트은 제목 밑에 리스트로 상세하게 
적혀있다.

각 `.html` 문서는 구조화되어 같은 정보를 담고 있는 `.json`과 대응된다. 이 기능은 
실험적이고 문서를 프로그래밍 적인 것으로 다루는 IDE나 다른 유틸리티의 이점을 위해서 
추가되었다.

각 `.html`과 `.json`파일은 node의 소스트리에서 `doc/api/` 폴더 안에 
대응되는 `.markdown` 파일로 생성된다. 이 문서는 `tools/doc/generate.js` 
프로그램을 사용해서 생성한다. HTML 템플릿은 `doc/template.html`에 있다.

## Stability Index

<!--type=misc-->

Throughout the documentation, you will see indications of a section's
stability.  The Node.js API is still somewhat changing, and as it
matures, certain parts are more reliable than others.  Some are so
proven, and so relied upon, that they are unlikely to ever change at
all.  Others are brand new and experimental, or known to be hazardous
and in the process of being redesigned.

The notices look like this:

    Stability: 1 Experimental

The stability indices are as follows:

* **0 - Deprecated**  This feature is known to be problematic, and changes are
planned.  Do not rely on it.  Use of the feature may cause warnings.  Backwards
compatibility should not be expected.

* **1 - Experimental**  This feature was introduced recently, and may change
or be removed in future versions.  Please try it out and provide feedback.
If it addresses a use-case that is important to you, tell the node core team.

* **2 - Unstable**  The API is in the process of settling, but has not yet had
sufficient real-world testing to be considered stable. Backwards-compatibility
will be maintained if reasonable.

* **3 - Stable**  The API has proven satisfactory, but cleanup in the underlying
code may cause minor changes.  Backwards-compatibility is guaranteed.

* **4 - API Frozen**  This API has been tested extensively in production and is
unlikely to ever have to change.

* **5 - Locked**  Unless serious bugs are found, this code will not ever
change.  Please do not suggest changes in this area; they will be refused.

<!--type=misc-->

문서 전체에서 섹션의 안정성 지표를 볼 것이다. Node.js API는 아직도 조금씩 바뀌고 
있고 성숙도에 따라 어떤 부분은 다른 부분보다 더 신뢰할 수 있다. 어떤 것들은 그렇게
증명되었고 전혀 변경될 가능성이 없어 보이는 것에 의존하고 있다. 다른 것들은 새롭고
실험적이거나 위험하다고 알려져 있거나 새로 디자인하고 있다.

이 표시는 다음과 같다:

    안정성: 1 Experimental

안정성 지표는 다음과 같다:

* **0 - Deprecated**  이 기능은 문제의 소지가 있는 것으로 알려졌고 변경될 예정이다.
여기에 의존하지 말아라. 이 기능을 사용하면 경고나 나올 것이다. 하위호환성을 기대하지 말아야
한다.

* **1 - Experimental**  이 기능은 최근에 도입되었고 차기 버전에서 변경되거나 제거될 것이다.
이 기능을 시험해보고 피드백을 주기 바란다. 이 기능이 당신에게 중요한 유즈케이스를 제공한다면
노드 코어팀에게 말해줘라.

* **2 - Unstable**  API가 안정화되는 과정에 있지만 아직 안정적이라고 고려될 만큼 충분한
실세계의 테스트를 거치지 않았다. 합당하다면 하위호환성은 유지될 것이다.

* **3 - Stable**  API가 충분히 검증되었지만 기반이 되는 코드의 정리때문에 마이너한 변경이
있을 수 있다. 하위호완성이 보장된다.

* **4 - API Frozen**  이 API는 프로덕션레벨에서 광범위하게 테스트되었고 변경될 여지가 
거의 없다.

* **5 - Locked**  심각한 버그가 발견되지 않는한 이 코드는 절대 변경되지 않을 것이다. 이 
영역에서 변경사항을 제안하지 말아라. 아마도 거절될 것이다.

## JSON Output

    Stability: 1 - Experimental

Every HTML file in the markdown has a corresponding JSON file with the
same data.

This feature is new as of node v0.6.12.  It is experimental.

    안정성: 1 - Experimental

마크다운의 모든 HTML파일은 같은 데이터를 가진 대응되는 JSON 파일이 있다.

이 기능은 node v0.6.12에서 추가되었고 실험적인 것이다.
