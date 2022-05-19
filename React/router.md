# React Router
## Router
```javascript
import React from "react";
import {
  // HashRouter as Router, => 잘 사용 안함
  BrowserRouter as Router,
  Routes,
  Route,
} from "react-router-dom";
import Detail from "./routes/Detail";
import Home from "./routes/Home";

function App() {
  return (<Router>
    {/*Route가 한번에 두개를 렌더링 할 수도 있기 때문에 한번에 하나만 렌더링 하려면 Routes를 넣어야한다.*/}
    <Routes>
      <Route path="/movie/:id" element={<Detail/>} />
      <Route path="/" element={<Home/>} />
    </Routes>
  </Router>);
}

export default App;
```
```javascript
import {Link} from "react-router-dom";

function Movie({id, coverImg, title, summary, genres}) {
    return (
        <div>
            <img src={coverImg} alt=""></img>
            <h2>
                <Link to={`/movie/${id}`}>{title}</Link>
            </h2>
            <p>{summary}</p>
            <ul>
                {genres.map(g => <li key={g}>{g}</li>)}
            </ul>
        </div>
    );
}
```
* Router는 어느 url에서 어느 컴포넌트를 렌더링 할 지 결정할 때 사용한다.
* 특정 url로 링크를 걸어줄 때는 Link 태그를 사용한다. a태그를 사용해도 링크가 걸리긴 하지만 페이지 전체가 다시 로딩되기 때문에 속도면에서 좋지 않다. 하지만 Link 태그를 사용하면 페이지 전체를 로딩하는 대신 화면만 다시 렌더링된다.
## PathVariable
```javascript
<Route path="/movie/:id">
    <Detail/>
</Route>
```
```javascript
<Movie
    key={movie.id} 
    {/*Movie 함수의 파라미터로 id 값 전달*/}
    id={movie.id} 
    coverImg={movie.medium_cover_image}
    title={movie.title}
    summary={movie.summary}
    genres={movie.genres}>
</Movie>
```
```javascript
//Movie function -> 파라미터로 id 받음
<Link to={`/movie/${id}`}>{title}</Link>
```
```javascript
// "/movie/:id"로 라우팅 시 실행
import { useEffect, useState } from "react";
import { useParams } from "react-router-dom";

function Detail() {
    const [loading, setLoading] = useState(true);
    const [movie, setMovie] = useState({});
    
    //PathVariable 추출
    const {id} = useParams();
    const getMovie = async () => {
        const response = await fetch(`https://yts.mx/api/v2/movie_details.json?movie_id=${id}`);
        const json = await response.json();
        setLoading(false);
        setMovie(json);
        console.log(json);
    }
    useEffect(() => {
        getMovie();
    }, [])
    return (
        <div>
            {loading ? (
        <h1>loading...</h1>
        ) : (<div>
                <h1>{movie.data.movie.title}</h1>
                <img src={movie.data.movie.small_cover_image}/>
                <ul>
                    {movie.data.movie.genres.map((genre, idx) => <li key={idx}>{genre}</li>)}
                </ul>
                <p>{movie.data.movie.description_full}</p>
            </div>)}
        </div>
    );
}
```
* useParams()를 사용하면 url에서 PathVariable을 추출할 수 있다. 이때 변수 명은 `<Route path="/movie/:id">`에서 지정한 변수명으로 해야한다.