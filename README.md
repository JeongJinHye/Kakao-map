# Kakao-map

<img src="https://user-images.githubusercontent.com/109572328/208230782-6f5b6e6d-3c83-4580-8a74-ee276865833a.jpeg" />

<h3>사용한 언어</h3>
<ol>
  <li>React</li>
  <li>Typescript</li>
</ol>

<h3>구현 목표</h3>
<ol>
  <li>Maps API를 사용해 지도 가져오기</li>
  <li>Local API를 사용해 장소 정보 불러오기</li>
  <li>검색 리스트 & 리스트 순으로 마커 띄우기</li>
  <li>장소 정보 팝업으로 보여주기</li>
</ol>

<h3>제작 기간</h3>
<ul>
  <li>22.10.01 - 22.10.09</li>
</ul>

<h3>코드 정리</h3>
- KakaoMapScriptLoader 컴포넌트
<p>
많이 사용할 스크립트 아이디, 키를 환경 변수로 빼줬습니다.

```
const KAKAO_MAP_SCRIPT_ID = "kakao-map-script";
const KAKAO_MAP_APP_KEY = process.env.KAKAO_MAP_KEY;
```

props의 children에는 타입을 유연하게 정의해주는 ReactNode를 사용해서 타입을 잡아줬습니다.

```
interface KakaoMapScriptLoaderProps {
  children: ReactNode;
}
```

스크립트 로드에 성공했을 경우와 실패했을 경우를 관리해주기 위해 useState로 변수를 만들었습니다.

```
const [mapScriptLoaded, setMapScriptLoaded] = useState(false);
```

값이 변경되지 않는 이상 처음 한번만 생성되는 useEffect를 사용해 스크립트 태그를 만들어서 카카오 SDK를 넣어주고 로드되는 시점을 명시적으로 잡아줬습니다.<br>
예외 처리도 함께 해줬습니다.

```
useEffect(() => {
  const mapScript = document.getElementById(KAKAO_MAP_SCRIPT_ID);

  if (mapScript && !window.kakao) {
    return;
  }

  const script = document.createElement("script");
  script.id = KAKAO_MAP_SCRIPT_ID;
  script.src = `https://dapi.kakao.com/v2/maps/sdk.js?appkey=${KAKAO_MAP_APP_KEY}&libraries=services&autoload=false`;
  script.onload = () => {
    window.kakao.maps.load(() => {
      setMapScriptLoaded(true);
    });
  };
  script.onerror = () => {
    setMapScriptLoaded(false);
  };
}, []);
```
  
그리고 appendchild를 사용해서 root의 자식으로 script 태그를 넣어줬습니다.
  
```
document.getElementById("root")?.appendChild(script);
```
<p>
