# Kakao-map

<img src="https://user-images.githubusercontent.com/109572328/208230782-6f5b6e6d-3c83-4580-8a74-ee276865833a.jpeg" />

<h3>사용한 기술</h3>
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

- 지도 띄우기

```
// useState로 로드됐는지 확인해주는 변수를 만들어줬습니다.
const [mapScriptLoaded, setMapScriptLoaded] = useState(false);

// 함수 컴포넌트에 useEffect를 사용했습니다.
useEffect(() => {
    const mapScript = document.getElementById(KAKAO_MAP_SCRIPT_ID);

    // 스크립트 아이디가 이미 있거나 로딩되지 않을 경우의 예외처리까지 해줬습니다.
    if (mapScript && !window.kakao) {
      return;
    }
    
    // 스크립트 태그를 만들어 주고
    const script = document.createElement("script"); 
    script.id = KAKAO_MAP_SCRIPT_ID;
    script.src = `https://dapi.kakao.com/v2/maps/sdk.js?appkey=${KAKAO_MAP_APP_KEY}&libraries=services&autoload=false`;
    
    // 로드되는 시점을 명시적으로 잡아줬습니다.
    script.onload = () => {
      window.kakao.maps.load(() => {
        setMapScriptLoaded(true);
      });
    };
    script.onerror = () => {
      setMapScriptLoaded(false);
    };
    
    document.getElementById("root")?.appendChild(script); 
    // root의 하위 자식으로 스크립트 태그를 넣어줬습니다.
```

- 지도 움직이기

```
// 타입을 kakao.maps.Map으로 지정해줬습니다.
const [map, setMap] = useState<kakao.maps.Map>();

// useRef를 사용해서 특정 요소의 주소값을 담는 변수를 만들어줬습니다.
const kakaoMapRef = useRef(null);

  useEffect(() => {
    // ref의 예외처리를 해줬습니다.
    if (!kakaoMapRef.current) {
      return;
    }

    // 기본 주소는 카카오 제주이며 옵션은 각각 처음 지도 로드 시 중심이 되는 위치와 확대 레벨입니다.
    const targetPoint = new kakao.maps.LatLng(33.450701, 126.570667);
    const options = {
      center: targetPoint,
      level: 3,
    };

    // 첫번째 인자로 Map DOM을 두번째 인자로는 옵션을 넘겨줬습니다.
    setMap(new window.kakao.maps.Map(kakaoMapRef.current, options));
  }, []);

<Container>
  // Map 태그에 미리 할당했던 useRef 객체를 넣어줬습니다.
  <Map ref={kakaoMapRef} />
</Container>
{map ? (
  // 커스텀 훅을 이용해서 하위에 로드되는 children이 provider를 통해 map context에 접근할 수 있게 했습니다.
  <KakaoMapContext.Provider value={map}>
    {props.children}
  </KakaoMapContext.Provider>
) : (
  // 오류를 확인하기 위해 null 대신 div 태그를 사용했습니다.
  <div>지도 정보를 가져오는데 실패했습니다.</div>
)}
```

- 검색 리스트

```
// 커스텀 훅 받아오기
const map = useMap();

  // keyword에 사용자가 입력한 값을 이벤트를 통해 받아오기 위해 빈 문장을 넣어줬습니다.
  const [keyword, setKeyword] = useState("");
  // 검색 리스트에 장소를 표시해주기 위해 state로 관리해줬습니다.
  const [places, setPlaces] = useState<PlaceType[]>([]);
  const placeService = useRef<kakao.maps.services.Places | null>(null);

  useEffect(() => {
    // placeService가 존재하면 리턴하도록 하였고
    if (placeService.current) {
      return;
    }
    
    // 존재하지 않으면 placeService를 가져오게 하였습니다.
    placeService.current = new kakao.maps.services.Places();
  }, []);

  // 카카오 가이드 코드를 사용했습니다.
  const searchPlaces = (keyword: string) => {
    if (!keyword.replace(/\s+|\s+$/g, "")) {
      alert("키워드를 입력해주세요!");
      return;
    }

    // 오류가 발생해서 임시로 오류처리를 해줬습니다.
    if (!placeService.current) {
      alert("placeService 에러");
      return;
    }

    placeService.current.keywordSearch(keyword, (data, status) => {
    
      // 검색에 성공한 경우에 필요한 데이터만 표시해주고 setPlaces에 해당 info값을 저장하도록 했습니다.
      if (status === kakao.maps.services.Status.OK) {
        const placeInfos = data.map((placeSearchResultitem) => {
          return {
            id: placeSearchResultitem.id,
            position: new kakao.maps.LatLng(
              Number(placeSearchResultitem.y),
              Number(placeSearchResultitem.x)
            ),
            title: placeSearchResultitem.place_name,
            address: placeSearchResultitem.address_name,
          };
        });

        props.onUpdatePlaces(placeInfos);
        setPlaces(placeInfos);
      } else if (status === kakao.maps.services.Status.ZERO_RESULT) {
        alert("검색 결과가 존재하지 않습니다.");
        return;
      } else if (status === kakao.maps.services.Status.ERROR) {
        alert("검색 결과 중 오류가 발생했습니다.");
        return;
      }
    });
  };

  // Form 태그의 제출을 막아줬습니다.
  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    searchPlaces(keyword);
  };

  // 검색 장소를 클릭한 경우 해당 장소를 기준으로 중앙으로 이동하도록 했습니다.
  const handleItemClick = (place: PlaceType) => {
    map.setCenter(place.position);
    map.setLevel(4);
    props.onSelect(place.id);
  };
```

- 검색 장소에 마커 표시, 팝업 띄우기

```
const MapMarkerController = (props: MapMarkerControllerProps) => {
  const map = useMap();

  useEffect(() => {
    if (props.places.length < 1) {
      return;
    }
    
    // map에 LatLngBounds 객체를 추가하고
    const bounds = new window.kakao.maps.LatLngBounds();
    // 검색된 장소를 기준으로 지도 범위를 재설정하기 위해 LatLngBounds 객체에 좌표를 추가했습니다.
    props.places.forEach((place) => {
      bounds.extend(place.position);
    });

    // 그리고 bounds 값을 setBounds에 넣어줬습니다.
    map.setBounds(bounds);
  }, [props.places]);

const MARKER_IMAGE_URL =
  "https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/marker_number_blue.png";

const MapMarker = (props: MapMarkerProps) => {
  const map = useMap();
  const container = useRef(document.createElement("div"));

  // 마커와 마찬가지로 팝업에도 useMemo를 사용했습니다.
  const infoWindow = useMemo(() => {
    container.current.style.position = "absolute";
    container.current.style.bottom = "40px";

    return new kakao.maps.CustomOverlay({
      position: props.place.position,
      content: container.current,
    });
  }, []);

  console.log(props.showInfo);

  // 마커가 여러번 생성될 필요는 없어서 useMemo 사용
  const marker = useMemo(() => {
    const imageSize = new kakao.maps.Size(36, 37);
    const imgOptions = {
      spriteSize: new kakao.maps.Size(36, 691),
      spriteOrigin: new kakao.maps.Point(0, props.index * 46 + 10),
      offset: new kakao.maps.Point(13, 37),
    };
    const markerImage = new kakao.maps.MarkerImage(
      MARKER_IMAGE_URL,
      imageSize,
      imgOptions
    );

    const marker = new kakao.maps.Marker({
      map: map,
      position: props.place.position,
      image: markerImage,
    });

    // 마커 클릭시 검색 장소 기준 중앙으로 이동
    kakao.maps.event.addListener(marker, "click", function () {
      map.setCenter(props.place.position);
      map.setLevel(4, {
        animate: true, // 부드럽게 움직이기
      });
      infoWindow.setMap(map);
    });

    return marker;
  }, []);

  useLayoutEffect(() => {
    // 지도 위에 마커 표시
    marker.setMap(map);

    return () => {
      // 마커 제거
      marker.setMap(null);
    };
  }, [map]);

  useEffect(() => {
    if (props.showInfo) {
      infoWindow.setMap(map);
      return;
    }
    return () => {
      // 컴포넌트가 언마운트될때 setMap의 값을 null로 넣어줘서 팝업이 삭제되도록 했습니다.
      infoWindow.setMap(null);
    };
  }, [props.showInfo]);

  return container.current
    ? ReactDOM.createPortal(
        <Message
          onClick={() => {
            infoWindow.setMap(null);
          }}
        >
          <Title>{props.place.title}</Title>
          <Address>{props.place.address}</Address>
        </Message>,
        container.current
      )
    : null;
};
```
