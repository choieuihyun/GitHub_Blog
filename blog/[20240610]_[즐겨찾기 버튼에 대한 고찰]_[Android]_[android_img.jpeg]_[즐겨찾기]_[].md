# 즐겨찾기 기능 구현에 대한 고찰

## SharedPreference VS RoomDB
> 내가 생각하고 아는대로 적는 고찰  
> 남들처럼 길게 정리 못하겠다.

 **1. Room DB 이용**

뭐 다른 블로그 가서 글 보니까 이런 유형의 글이 있더라.

데이터 클래스 코드
```kotlin
  data class FoodDiaryModel ( // 실제 내 코드

  val id: Int = 0,
  val kcal: Float?,
  val foodName: String?,
  var buttonState: Boolean

) {

  fun toggleFavorite() {
    buttonState != buttonState
  }
}
```
Room DB에 데이터 Insert 코드

```kotlin
recyclerViewAdapter.setOnFavoriteButtonClickListener { // 실제 내 코드
  val foodDiary = FoodDiaryModel(
    kcal = it.nUTRCONT1?.toFloat(),
    foodName = it.dESCKOR,
    buttonState = false
  )
  foodDiary.toggleFavorite()
  viewModel.addFoodDiary(foodDiary)
}
```
위의 두 코드를 보면 리스너 내부에서 Model을 생성 후 클릭 시 추가하는 구조라고 유추 가능하다.  
즐겨찾기에 저장 시 데이터의 이동 경로를 보면 외부 API -> 앱 -> 로컬 DB의 구조를 띄는데 어댑터 파라미터로 전달되는 데이터는 외부 API를 통해 받아오는 데이터이기 때문에 buttonState에 대한 값이 없다.  
코드에는 false로 초기화 해놨지만 제대로 구현한다면 viewModel에 상태 변수를 만들어두고 그에 따른 제어를 해야할 것이라 생각한다.  
> **여기서 의문점이 하나 생긴다. 굳이 이렇게 해야하나? 머리 아픈데?** 

 **2. SharedPreference 이용**

내가 즐겨찾기 기능 구현할 때 버튼 상태를 SharedPreference로 구현하는 가장 중요한 이유 하나가 있다. **(뇌피셜 주의)**  

![이미지](../img/kcal_list.png)

여기서 보면 검색과 즐겨찾기의 RecyclerView가 같은 데이터 리스트를 가지는가? 아니다.  
생각의 순서로 다시 나열해보자.  

1. 검색은 외부 API를 네트워크 통신을 통해 데이터를 받아와 리스트로 표시, 즐겨찾기는 로컬 DB에서 데이터를 받아와 리스트로 표시한다.  
2. 그러면 검색 리스트에서 즐겨찾기 버튼을 눌렀다고 할 때, 검색 리스트와 즐겨찾기 리스트의 버튼 클릭 상태는 동시에 변경되어야 한다.  
3. 이는 반대로 즐겨찾기 리스트에서 즐겨찾기 버튼을 눌렀을 때, 검색 리스트의 아이템도 즐겨찾기 버튼의 상태에 변화가 있어야 한다는 것이다.  

이러한 전개의 결과를 생각해보자. 데이터 클래스 내부에 클릭 상태를 저장한다면 네트워크 통신을 통해 불러온 데이터 리스트와 로컬에서 불러온 데이터 리스트를 항상 비교해 버튼의 클릭 상태를 결정할 것인가?  

비효율적인 절차라고 생각한다. 그럴 바에 데이터 키 값(여기서는 음식 명)이 겹치지 않는 것을 확인했으니 SharedPreference를 사용하여 음식 명을 key로 사용하고, value에 상태를 저장하면 되지 않는가?  

말이 너무 길어서 이해가 안된다면 다르지만 간단하게 생각해볼 수도 있다.  
1. 검색 리스트와 즐겨찾기 리스트는 다른 데이터 리스트를 사용함에도 즐겨찾기 버튼은 같은 상태를 가져야하고, 상태에 따라 클릭 이벤트가 결정된다.
2. 상태는 작은 데이터이기 때문에 SharedPreference를 쓰기 적합하고, 이를 사용한다면 굳이 다른 비교나 동작이 추가될 일이 없음.