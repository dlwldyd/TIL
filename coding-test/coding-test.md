# 코딩테스트
## LIS 알고리즘
```c++
// vector<int> v 에는 최장 증가 부분 수열을 구할 배열 값이 들어있다.
// vector<int> length의 length[i] 에는 v의 인덱스 i에서 끝나는 최장 증가 부분 수열의 길이가 들어간다.
// 처음 vector<int> length는 전부 1로 초기화 되어있다.
for(int i=1; i<v.size(); i++){
    for(int j=0; j<i; j++){
        if(v[j] < v[i]){
            if(length[j] + 1 > length[i]){
                length[i] = length[j] + 1;
            }
        }
    }
}
```
* LIS 알고리즘은 DP를 이용한 알고리즘이다.
* 첫 번째 반복문에서는 인덱스 i에서 끝나는 최장 증가 부분 수열을 구하기위해 1부터 v.size()까지 반복한다.(인덱스 1부터 인덱스 v.size()-1 까지 각각의 인덱스에서 끝나는 최장 증가 부분 수열을 모두 구한다.)
* 두 번째 반복문에서는 만약 v[j] 값이 v[i] 값보다 크다면 `{인덱스 j에서 끝나는 최장 증가 부분 수열, v[i]}` 도 증가하는 부분 수열이므로 그 중에서 가장 긴 수열을 구하기 위해 length[j] + 1 > length[i] 인 경우에만 length[i] = length[j] + 1 를 한다.
## 구현
### 각 자리수의 값 구하기
```c++
int n = 23485;
for(int i=1; i=<n; i*=10){
    cout << n / i % 10 << " ";
}
```
* `5 8 4 3 2`출력 됨
### 위, 아래, 좌, 우 이동
```c++
int dx[] = {0, 0, -1, 1};
int dy[] = {1, -1, 0, 0};
```
* 이런식으로 방향을 배열로 정의해두면 편하다.
* 0 -> 아래, 1 -> 위, 2 -> 좌, 3 -> 우
### 조합
```c++
#include <bits/stdc++.h>

using namespace std;

int n, m;

//배열 num의 시작지점(s)에서 끝까지 중 r개만큼을 선택
void select(int s, int r, int *num, string p){
    if(r == 0){
        cout << p;
        return;
    }
    //i는 무조건 선택, i를 선택했기 때문에 r-1개 만큼 더 선택해야 하므로 i는 n-(r-1)까지만 순환 가능
    for(int i=s; i<n-r+1; i++){
        p[(m-r)*2] = num[i] + '0';
        select(i+1, r-1, num, p);        
    }
}

int main(){

    cin.tie(NULL);
    cout.tie(NULL);
    ios::sync_with_stdio(false);

    cin >> n >> m;
    int num[n];
    string p;
    for(int i=0; i<m; i++){
        p.append("  ");
    }
    p.append("\n");
    for(int i=0; i<n; i++){
        num[i] = i+1;
    }
    select(0, m, num, p);
}
```
* 조합은 재귀함수를 통해서 얻을 수 있다.
### 잡다
* `cin.eof()` : 입력의 종료 조건이 없을 때 파일 끝까지 입력받기 위해 쓰자
* `cin.tie(NULL)`, `cout.tie(NULL)`, `ios::sync_with_stdio(false)` : 출력버퍼에 저장했다가 한번에 출력할 때, 입출력 속도가 빠를 필요가 있을 때 사용하자. 단, 이걸 사용하려면 `endl` 대신에 `\n`를 사용하자
* `cout.precision(5)`, `cout << fixed` : 소수점 출력 자리수 정할 때, `cout << fixed` 사용안하면 소수점 자리수를 `cout.precision(5)`로 5로 정했다 해도 소수점 아래자리수가 0이면 출력 안함 예를 들면 0.33400 -> 0.334까지만 출력
* 규칙적인 패턴을 가진 문양과 같은 것에 대한 문제는 수열을 가지고 쉽게 풀 수 도 있다.
* 일반적인 직교 좌표계는 이차원 배열과 비교해서 x방향은 같지만 y방향은 반대이다.
* 반올림 : `round(doube)`, 올림 :  `ceil(double)`, 내림 : `floor(double)`. 소수점 첫째자리에서 반올림, 올림, 내림되며 반환값을 double이다. 만약 소수점 첫째자리가 아니라 다른 자리수에서 반올림, 올림, 내림을 하고 싶으면 10<sup>n</sup> 만큼 곱하거나 나눈 뒤 함수를 적용한 후 다시 반대로 나누거나 곱하면 된다.