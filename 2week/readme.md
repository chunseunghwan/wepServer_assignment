# [Web 실습 추가 학습 보고서] 멜론 차트 페이지 - 가수별 필터링 기능 구현

**작성자**: 천승환 (20231161)
**관련 실습**: Week 02 실습 - 멜론 차트 HTML/CSS 페이지
**학습 형태**: 강의 실습 코드 기반 개인 추가 학습
**사용된 AI**: 클로드 생성형 대화

---

## 1. 개요

Week 02 실습에서 작성한 "멜론 차트" 페이지는 노래 제목과 이미지를 카드 형태로 나열하고, 클릭 시 유튜브 검색 결과로 이동하는 정적인 HTML/CSS 페이지였다. 이 페이지는 노래 목록이 하나로 나열되어 있을 뿐, 가수 정보나 가수별 분류 기능은 없었다.

강의 내용을 복습하면서 "실제 음악 서비스처럼 가수를 선택하면 그 가수의 곡만 보이게 할 수는 없을까?" 라는 궁금증이 생겨, 기존 실습 코드를 바탕으로 JavaScript를 추가로 학습하고 가수별 필터링 기능을 직접 구현해보았다.

대부분의 과정에서 생성형 인공지능의 도움이 있었고 인공지능이 만들어준 파일을 기반으로 학습을 진행함

## 2. 학습 목표

- 정적 HTML(하드코딩된 마크업)을 데이터 기반(JavaScript 배열 + 동적 렌더링) 구조로 바꾸는 방법 이해
- JavaScript의 배열 메서드(`map`, `filter`)와 `Set`을 이용한 중복 제거 활용법 학습
- DOM을 직접 조작하는 방법(`createElement`, `innerHTML`, `appendChild`) 학습
- 이벤트 리스너(`addEventListener`)를 이용해 클릭 인터랙션 구현
- C/Java의 배열·객체 개념을 JavaScript의 객체 배열(배열 안에 `{key: value}` 구조)로 옮겨 생각해보는 연습

## 3. 기존 실습 코드의 구조와 한계

기존 코드는 아래와 같이 `.container` 블록을 노래 개수만큼 **손으로 복사·붙여넣기**해서 작성하는 방식이었다.

```html
<div class="container">
  <a href="https://www.youtube.com/results?search_query=노래방+My way" target="_blank">
    <img src="..." alt="랜덤 이미지 1">
    <div class="song-title">My way</div>
  </a>
</div>
```

이 방식은 노래가 3곡일 때는 문제없지만,
- 노래가 많아지면 같은 구조를 계속 복사해야 해서 HTML이 매우 길어지고,
- "가수"라는 정보 자체가 코드 어디에도 없어서, 가수별로 묶거나 필터링할 방법이 없다는 한계가 있었다.

C언어에서 배열 원소를 `printf`로 하나씩 출력하던 것과 비교하면, 기존 코드는 마치 출력 결과(HTML)만 미리 다 써놓은 것과 같고, "데이터"와 "화면에 보여주는 로직"이 분리되어 있지 않은 구조라고 이해했다.

## 4. 추가 구현 내용

### 4.1 데이터를 배열로 분리

가장 먼저 한 일은 노래 정보를 HTML에서 분리해 JavaScript 배열(`songs`)로 옮긴 것이다. 노래 하나하나를 C의 구조체(struct)와 비슷하게 `artist`, `title`, `query`, `img` 필드를 가진 객체로 표현했다.

```js
const songs = [
  { artist: "Frank Sinatra", title: "My Way", query: "노래방 My way", img: "..." },
  { artist: "Frank Sinatra", title: "Fly Me To The Moon", query: "노래방 Fly me to the moon", img: "..." },
  { artist: "Frank Sinatra", title: "That's Life", query: "노래방 That's life", img: "..." },
  { artist: "IU", title: "Good Day", query: "노래방 좋은날 아이유", img: "..." },
  { artist: "IU", title: "Through the Night", query: "노래방 밤편지 아이유", img: "..." }
];
```

기존 실습의 3곡(My Way, Fly Me To The Moon, That's Life)은 사실 전부 Frank Sinatra의 곡이라는 것을 이번에 찾아보며 알게 되었다. 필터링 기능이 실제로 동작하는 모습을 확인하려면 가수가 두 명 이상 있어야 하므로, 비교 대상으로 IU의 곡 2개를 예시 데이터로 추가했다.

### 4.2 가수 목록 자동 추출 (배열 → 중복 제거)

가수 탭 버튼을 하나씩 손으로 만드는 대신, `songs` 배열에서 가수 이름만 뽑아 자동으로 만들도록 구현했다.

```js
const artists = ["전체", ...new Set(songs.map(s => s.artist))];
```

- `songs.map(s => s.artist)` : 모든 곡에서 가수 이름만 뽑아 새로운 배열을 만든다.
- `new Set(...)` : 배열을 Set에 넣으면 중복된 값이 자동으로 제거된다 (Frank Sinatra가 3번 나와도 Set 안에는 1번만 남음).
- `[...Set]` : 다시 배열 형태로 풀어준다.
- 맨 앞에 `"전체"`를 추가해서 전체 곡을 보는 옵션도 넣었다.

이 부분에서 Set 자료구조를 처음 제대로 사용해봤는데, Java의 `HashSet`과 개념적으로 동일하다는 것을 알고 나니 이해가 훨씬 쉬웠다.

### 4.3 동적 렌더링 함수 작성

노래 카드를 그릴 때도 HTML을 직접 쓰지 않고, JavaScript가 `songs` 배열을 반복하면서 DOM 요소를 만들어 붙이도록 했다.

```js
function renderSongs(selectedArtist) {
  listEl.innerHTML = "";
  const filtered = selectedArtist === "전체"
    ? songs
    : songs.filter(s => s.artist === selectedArtist);

  filtered.forEach(song => {
    const card = document.createElement("div");
    card.className = "container";
    card.innerHTML = `
      <a href="https://www.youtube.com/results?search_query=${encodeURIComponent(song.query)}" target="_blank">
        <img src="${song.img}" alt="${song.title} 이미지">
        <div class="song-info">
          <div class="song-title">${song.title}</div>
          <div class="song-artist">${song.artist}</div>
        </div>
      </a>
    `;
    listEl.appendChild(card);
  });
}
```

핵심은 `Array.prototype.filter`이다. `selectedArtist`와 같은 `artist` 값을 가진 곡만 골라 새로운 배열(`filtered`)을 만들고, 그 배열만 화면에 그린다. C에서 `for`문을 돌며 `if (조건)` 일 때만 처리하던 방식과 원리는 같지만, JavaScript는 `filter` 한 줄로 그 과정을 대신해준다는 점이 인상적이었다.

`encodeURIComponent`는 검색어에 공백이나 특수문자가 있을 때 URL이 깨지지 않도록 안전하게 인코딩해주는 함수라는 것도 이번에 찾아보며 알게 되었다.

### 4.4 가수 탭 버튼과 클릭 이벤트

가수 탭 버튼도 `artists` 배열을 기반으로 자동 생성하고, 각 버튼에 클릭 이벤트를 등록했다.

```js
function renderTabs(selectedArtist) {
  tabsEl.innerHTML = "";
  artists.forEach(artist => {
    const btn = document.createElement("button");
    btn.textContent = artist;
    if (artist === selectedArtist) btn.classList.add("active");
    btn.addEventListener("click", () => {
      renderTabs(artist);
      renderSongs(artist);
    });
    tabsEl.appendChild(btn);
  });
}
```

버튼을 클릭하면 `renderTabs`와 `renderSongs`를 그 가수 이름으로 다시 호출해서, 선택된 탭은 `active` 클래스로 강조되고 노래 목록도 그 가수 것만 다시 그려지도록 했다. 처음에는 클릭할 때마다 페이지가 새로고침되는 줄 알았는데, `addEventListener`로 등록한 함수는 페이지 이동 없이 DOM만 다시 그리는 것이라는 점을 이번에 명확히 이해했다.

### 4.5 CSS 보완

가수 이름을 보여줄 공간이 없어서, 기존 `.song-title`만 있던 카드 안쪽에 `.song-info`, `.song-artist` 클래스를 추가하고, 선택된 가수 탭을 초록색으로 강조하는 `.artist-tabs button.active` 스타일을 새로 작성했다. 기존 실습 코드의 색상(`#00cd3c`)과 카드 스타일(둥근 모서리, 그림자, hover 시 위로 뜨는 효과)은 그대로 유지해서 디자인 통일성을 지켰다.

## 5. 실행 결과

- 처음 페이지를 열면 "전체" 탭이 선택된 상태로 5곡(Frank Sinatra 3곡 + IU 2곡)이 모두 보인다.
- "Frank Sinatra" 탭을 클릭하면 My Way, Fly Me To The Moon, That's Life 3곡만 표시된다.
- "IU" 탭을 클릭하면 Good Day, Through the Night 2곡만 표시된다.
- 각 카드는 기존과 동일하게 클릭 시 새 탭에서 해당 곡의 유튜브 검색 결과로 이동한다.

## 6. 배운 점 및 느낀점

- C/Java에서 익숙했던 "배열 + 반복문 + 조건문" 사고방식이 JavaScript에서는 `map`, `filter`, `Set` 같은 배열 메서드로 더 간결하게 표현된다는 것을 체감했다.
- HTML을 미리 다 써두는 정적인 방식보다, 데이터(배열)와 화면을 그리는 함수를 분리해두면 데이터만 추가/수정해도 화면이 자동으로 바뀐다는 점이 훨씬 효율적이라는 것을 알게 되었다.
- `addEventListener`를 이용한 이벤트 기반 프로그래밍은 절차적으로 위에서 아래로 실행되는 C 코드와는 다른 흐름이라, 처음에는 실행 순서를 이해하는 데 시간이 걸렸다.
- 아직 DB 연동이나 서버 통신은 다루지 않았기 때문에, 다음 단계로는 실제 가수/곡 데이터를 서버(API)에서 받아와 표시하는 방식도 공부해보고 싶다.

## 7. 참고

- 이 보고서에서 다룬 코드는 실습 시간에 작성한 `Week 02` 멜론 차트 페이지 실습을 기반으로, 가수별 필터링 기능을 개인적으로 추가 구현한 것이다.
