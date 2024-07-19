---
emoji: ''
title: '컴포넌트 추상화'
date: '2024-04-02 22:00:00'
author: seoyul
tags: FrontEnd
categories: FrontEnd
---

# 컴포넌트 추상화

공통적인 요소들을 추출해 불필요한 부분들을 생략하고 객체의 속성 중 중요한 것에만 중점을 두어 개략화(사전 : 대강 추리는 일) 하는 것

## 하나의 컴포넌트가 하나의 책임만을 담당한다.

- 책임이란 어플리케이션에서 컴포넌트가 하는 행동
- 컴포넌트들을 각각의 책임에 맞게 필요한 로직만으로 구성한다면 응집도가 높아지며, 다른 컴포넌트와의 결합도는 줄고 컴포넌트 특성상 외부로 노출할 필요가 없는 부분은 캡슐화가 될 것
- 컴포넌트는 이벤트 기반으로 스타일과 DOM 구조까지 고려하여 고유한 책임을 수행하게 되기에 버튼에 스타일을 적용하거나 클릭했을 때의 애니메이션을 넣거나 필요에 따라 색상과 폰트를 변경하거나 생각보다 다양한 것들이 책임을 수행하기 위해 고려되어야 한다.
- 텍스트와 같이 작은 책임 수행을 넘어서 레이아웃의 형태를 책임지는 컴포넌트, 서버와의 통신을 통해 데이터를 가져오는 컴포넌트, 특정 도메인에 관련된 비즈니스 로직을 수행하는 컴포넌트 등 범위가 확대될수록 이러한 책임을 나누는 것은 점점 더 어려워지게 된다.

## 범용적 컴포넌트

- 모듈단위의 컴포넌트로 특정 도메인에 종속되지 않는다.
- 특정 도메인에만 필요한 props를 추가하지 않는다.
- 특정 도메인에만 필요한 분기처리를 하지 않는다.

## 특정 도메인에 종속된 컴포넌트

- 재사용성은 있으나 특정 도메인에 종속되어 범용적으로 사용하기 어려운 컴포넌트
- 합성 컴포넌트를 도입하여 도메인과 컴포넌트의 책임을 세분화 하여 의존성을 낮춘다.
- 너무 많은 컴포넌트가 도메인에 얽히지 않도록 주의

### Hook

- **기능 별**로 custom hook을 만들어 비즈니스 로직을 추상화 한다.
- 훅을 통한 책임의 분리는 컴포넌트의 응집도를 높이고 도메인과의 결합을 낮춘다.
- 예시
    
    ```jsx
    const AlbumList = ({ artistId }: Props) => {
      const [albums, setAlbums] = useState([]);
      const [dataLoaded, setDataLoaded] = useState(false);
      const ref = useRef<HTMLDivElement>(null);
      const mountedRef = useRef(true);
      
      useEffect(() => {
        const fetchAlbums = async () => {  
          const response = await fetch('/getAlbums');
          const data = await response.json();
    
          setAlbums(data);
          setDataLoaded(true);
        }
        fetchAlbums();
      }, []);
    
      useEffect(() => {
        // 뒤로 가기로 페이지를 이동하여 앨범들을 보여줄 경우 마지막 스크롤 위치를 계산하여 맞춰줍니다.
        if (mountedRef.current && dataLoaded) {
          const scrollTop = ...
          ref.current.scrollTop = scrollTop;
          mountedRef.current = false;
        }
      }, [dataLoaded]);
    
      return (
        <div ref={ref}>
          {albums.map(album => (
            <SquareCardItemList ... />
          ))}
        </div>
      );
    }
    ```
    
    - 해당 코드는 두 기능을 수행한 코드가 모두 컴포넌트에 작성되어 가독성이 떨어짐
    - 해당 UI를 다른 도메인에서도 사용하고 싶다면?
    - 해당 로직을 다른 도메인에서도 사용하고 싶다면?
    
    ```jsx
    const useFetchAlbums = (artistId: string) => {
      const [albums, setAlbums] = useState([]);
      const [dataLoaded, setDataLoaded] = useState(false);
      
      useEffect(() => {
        const fetchAlbums = async () => {  
          const response = await fetch('/getAlbums');
          const data = await response.json();
    
          setAlbums(data);
          setDataLoaded(true);
        }
        fetchAlbums();
      }, []);
    
      return [albums, dataLoaded]
    }
    
    const useLatestScrollTop = ({ ref, dataLoaded }: Props) => {
      const mountedRef = useRef(true);
    
      useEffect(() => {
        // 뒤로 가기로 페이지를 이동하여 앨범들을 보여줄 경우 마지막 스크롤 위치를 계산하여 맞춰줍니다.
        if (mountedRef.current && dataLoaded) {
          const scrollTop = ...
          ref.current.scrollTop = scrollTop;
          mountedRef.current = false;
        }
      }, [dataLoaded]);
    }
    
    const AlbumList = ({ artistId }: Props) => {
      const [albums, dataLoaded] = useFetchAlbums(artistId);
      const ref = useRef<HTMLDivElement>(null);
    
      useLatestScrollTop({ ref, dataLoaded });
      
      return (
        <div ref={ref}>
          {albums.map(album => (
            <SquareCardItemList ... />
          ))}
        </div>
      );
    }
    ```
    
    - 두가지 hook으로 각각의 기능을 추상화 한다.
    - 로직들을 훅으로 분리하여 컴포넌트에는 렌더링을 위한 코드만 응집되고 가독성이 좋아진다.
    - 해당 로직을 재사용할 수 있고, 각각을 단위테스트하여 검증하기도 좋다.
- hook을 설계할 때 가장 유의해야 할 점은 하나의 hook이 어떤 역할을 할 것인지, 중요한 역할이 무엇인지 명확하게 드러나게 설계해야한다.
- 복잡한 함수는 custom hook 안에 숨기고, hook의 중요한 역할을 드러내어 선언형 프로그래밍으로 작성해야 한다.

> 선언형 프로그래밍이란
> 
> - 핵심 데이터만 전달받고 세부 구현은 뭉쳐 숨겨 두는 개발 스타일
> - ‘무엇’을 하는 함수인지 빠르게 인지 가능
> - 세부 구현은 내부에 뭉쳐둠
> - ‘무엇’만 바꿔서 쉽게 재사용 가능
> - 함수가 어떤 방식으로 동작하는지보다 어떤 결과를 나타내는지에 중점
- 가능한 모든 코드를 뭉쳐서 하나의 커스텀 훅으로 묶어 배치하는 것이 좋은 것이 아니라, **해당 컴포넌트가 어떤 데이터를 어떻게 렌더링 하고 있는지 파악**을 할 수 있도록 작성하는것이 더 좋은 방법으로 어떻게 동작하는지 알 필요가 없는 부분만 커스텀 훅으로 숨겨두는 것

### 상태관리

- 상태관리 라이브러리를 사용시 **데이터간의 의존성을 낮추는 것이 중요하다.**
- 데이터가 서로 강하게 얽혀있거나 불필요한 정보가 포함되어 있다면 훅이나 상태 관리 라이브러리를 사용하여 로직을 분리하기 어렵고 재사용성도 떨어진다.
- 데이터 간의 의존성 때문에 컴포넌트에서는 상태 변경을 위한 코드 처리가 늘어나고, 데이터 처리를 위한 로직들이 Redux뿐만 아니라 컴포넌트와 훅으로 여기저기 흝어져 응집도가 떨어질 수 있다.
    - 로직을 Redux 내부로 캡슐화하여 컴포넌트와의 결합도를 더욱 낮춘다.
    - 데이터간의 서로 강하게 의존성이 있다면 데이터 구조를 통합하여 해결한다.
    - 어플리케이션의 도메인이 복잡할수록 데이터간의 의존성을 최소화하는 추상화가 중요
- 예시
    
    ```jsx
    // 재생 중인 곡에 대한 상태 관리
    /**
     * playingSong: 재생 중인 곡의 메타 정보(곡, 앨범, 아티스트 명).
     * playingSongLyrics: 재생 중인 곡의 가사.
     * seek: 현재 재생 중인 곡의 재생시간.
     * playtime: 재생 중인 곡의 전체 재생시간.
     */
    interface SongState {
      playingSong: Song | null;
      playingSongLyrics: Lyrics | null;
      seek: number;
      playtime: number;
    }
    
    setPlayingSong(song) {
      state.songs = song;
    }
    
    // 재생목록 상태 관리
    /**
     * songs: 곡들의 id를 저장하고 있는 재생목록.
     * playingSongIndex: 현재 재생 중인 곡의 index. 
     * repeatType: 반복 설정 상태.
     */
    interface SongListState {
      songs: number[];
      playingSongIndex: number;
      repeatType: RepeatType;
    }
    
    setSongs(songs) {
      state.songs = songs;
    }
    
    setPlayingSongIndex(index) {
      state.playingSongIndex = index;
    }
    
    // 컴포넌트에서 재생목록 갱신 코드
    const songs = [...];
    
    dispatch(setSongs(songs));
    
    if (songs.length) {
      const songId = songs[0];
      const { data } = await getSongDetailInfo(songId);
    
      dispatch(setPlayingSong(data));
      dispatch(setPlayingSongIndex(0));
    }
    ```
    
    ```jsx
    /**
     * playingSong: 재생 중인 곡의 메타 정보(곡, 앨범, 아티스트 명).
     * playingSongLyrics: 재생 중인 곡의 가사.
     * seek: 현재 재생 중인 곡의 재생시간.
     * playtime: 재생 중인 곡의 전체 재생시간.
     * songs: 곡들의 id를 저장하고 있는 재생목록.
     * playingSongIndex: 현재 재생 중인 곡의 index. 
     * repeatType: 반복 설정 상태.
     */
    interface SongState {
      songs: number[];
      playingSongIndex: number;
      repeatType: RepeatType;
      playingSong: Song | null;
      playingSongLyrics: Lyrics | null;
      seek: number;
      playtime: number;
    }
    
    // 컴포넌트에서 목록 갱신 코드
    const songs = [...];
    
    dispatch(setSongs(songs));
    
    if (songs.length) {
      dispatch(setPlayingSongIndex(0));
    }
    ```
    
    - 간단한 경우 통합할 수 있지만 각각이 큰 경우 오히려 거대해질 수 있음
    - 이러한 경우 각각 처리하도록 유지하고, 미들웨어나 전처리를 위한 별도의 인터페이스를 만들어 관리해 결합도는 유지하며 응집도를 높인다.
- 불필요한 결합도를 낮추기 위해 데이터를 통합할수도, 반대로 분리를 통해 결합도를 낮출수도 있다.
- 데이터 간의 강한 의존성이 있는지, 내부 구현의 캡슐화는 이뤄졌는지, 책임에 맞는 코드만 응집되었는지 끊임없이 스스로 질의하며, 변화에 유연한 코드를 작성해야 한다.

## 요약

- 범용 컴포넌트는 도메인 의존성을 제거하여 공통의 기능을 위한 응집도를 높이고 재사용성을 극대화해야 하며, 이 과정에서 너무 많은 책임을 수행하지 않도록 합성을 사용하여 적절하게 나누어야 한다.
- 도메인과 의존성이 있는 컴포넌트는 훅을 사용하여 컴포넌트와 비즈니스 로직의 결합도를 최대한 낮춰야 하며, 어플리케이션 전역에서 사용되는 도메인은 데이터 간의 의존성이 생기지 않도록 주의하여 설계해야 한다. 설계시 데이터를 통합하거나 분리하거나 여러 방법을 통해 모듈 간의 결합도를 낮추고 변화에 유연한 구조를 만드는 것이 중요하다.


### reference
[React 컴포넌트와 추상화](https://fe-developers.kakaoent.com/2022/221020-component-abstraction/)
[컴포넌트 추상화와 아키텍처 설계](https://velog.io/@xxziiko/%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EC%B6%94%EC%83%81%ED%99%94%EC%99%80-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EC%84%A4%EA%B3%84)