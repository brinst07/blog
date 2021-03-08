---
title: "RecyclerView"
date: 2021-01-11T23:51:11+09:00
author: "brinst"
draft: false
---

## 리사이클러뷰

- 리사이클러뷰는 스피너가 조금 더 확장된 형태이다.
- 리사이클러뷰도 스피너처럼 목록을 화면에 출력하는데, 레이아웃 매너지를 이용하면 간단한 코드만으로 일반 리스트뷰를 그리드뷰로 바꿀 수 도 있다.
- 리사이클러뷰처럼 목록을 표시하는 컨테이너들은 표시될 데이터와 아이템 레이아웃을 어댑터에서 연결해주므로 어댑터에서 어떤 아이템 레이아웃을 사용하느냐에 따라 표시되는 모양을 다르게 만들 수 있다.

### 매커니즘

1. 원하는 액티비티에 리사이클러뷰 만들기
2. 리사이클러뷰에 들어갈 item layout만들기
3. 리사이클러에 뿌릴 Model(dto) 만들기
4. 어댑터 정의하기

   - 리사이클러뷰는 리사이클러 어댑터라는 메서드 어댑터를 사용해서 데이터를 연결한다.
   - 리사이클러뷰 어댑터는 개별 데이터에 대응하는 뷰홀더 클래스를 사용한다.
   - 리사이클러뷰처럼 목록을 표시하는 컨테이너들은 표시될 데이터와 아이템 레이아웃을 어댑터에서 연결해주므로, 어댑터에서 어떤 아이템 레이아웃을 사용하느냐에 따라 표시되는 모양을 다르게 만들수 있다.

### 리사이클러 뷰 만들기

리사이클러뷰를 만들어준다. xml의 좌측 컨테이너를 클릭하면 RecyclerView를 확인 할 수 있는데, 이를 클릭하면 쉽게 리사이클러뷰를 추가할수 있다. 나는 ConstraintLayout 안에 RecylcerView를 넣었다.

### Itemlayout만들기

스피너와 마찬가지로 리사이클러뷰안에 들어가는 Item을 정의해줘야한다. 이때 디자인이나 형태를 정할 수 있다.
[app]-[res]-[layout] 디렉토리로 들어가 Layout resource file을 만들어준다.
만들떄 Root element를 LinearLayout으로 설정해준다.
그 후 본인이 원하는 방식으로 item을 만들어준다.

### Model 만들기

Item Layout을 만들었다면, item에 들어갈 Model을 만들어줘야한다.
data class를 사용하여 만들었다.

#### data class

코틀린은 간단한 값의 저장용도로 데이터 클래스를 제공한다.
data 예약어를 class 앞에 붙여서 만들고 클래스명 다음에 생성자처럼 파라미터를 정의할 수 있는데, 생성자와 다르게 파라미터 앞에 var 또는 val을 명시해서 변수인지 상수인지를 구분해야한다.

```Kotlin
data class 클래스명 (val 파라미터1 : 타입, var 파라미터2 : 타입)
```

또한 data class는 특별한 기능이 존재하는데, 일반 클래스에서 toString() 메서드를 호출하면 인스턴스의 주소값을 반환하지만,
data class는 값을 반환하기 대문에 실제값을 모니터링 할 떄 좋다.
copy() 메소드를 지원하기 때문에 간단하게 값을 복사하기에도 좋다.

위처럼 다양한 장점이 있는 data class로 Model을 만들었다.

```Kotlin
  data class Board (var no:Int, var title:String,var timestamp: Long)
```

### CustomAdapter 만들기

스피너는 이미 만들어진 Adapter를 사용했지만, RecyclerView는 직접 CustomAdapter를 만들어줘야한다.
리사이클러뷰 어댑터는 개별 데이터에 대응하는 뷰홀더 클래스를 사용한다.
상속하는 리사이클러뷰 어댑터에 뷰홀더 클래스를 제너릭으로 지정해야 하므로 뷰홀더 클래스를 먼저 만들고 나서, 어댑터 클래스를 생성하는 것이 더 편하다.

뷰홀더는 현재 화면에 보여지는 개수만큼 생성되고 목록이 위쪽으로 스크롤될 경우 가장 위 뷰홀더를 아래에서 재사용한 후 데이터만 바꿔주기 때문에 앱의 효율이 향상된다.(그래서 리사이클러... 재사용....)

```Kotlin
  //리사이클러 뷰가 동작하려면 아래의 3가지 메소드를 반드시 구현해야한다.
  class CustomAdapter{

    var listData = mutableListOf<Board>()

    //아이템 레이아웃을 생성하는 메소드
    //프로그램을 실행하면 한화면에 보이는 개수만큼 안드로이드가 호출한다.
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): Holder {
        //viewbinding을 사용하기 위한 코드
        val binding = ItemRecyclerBinding.inflate(LayoutInflater.from(parent.context),parent,false)
        return Holder(binding)
    }
    //스크롤이 될때마다 실제 화면에 데이터와 레이아웃을 연결하는 메서드
    override fun onBindViewHolder(holder: Holder, position: Int) {
        val board = listData.get(position)
        holder.setBoard(board)
    }

    override fun getItemCount(): Int {
        return listData.size
    }

  }

//어댑터에서 넘겨주는 아이템 레이아웃을 Holder의 클래스의 생성자에게서 받아 viewHolder의 생성자에게로 넘겨주는 구조
class Holder(val binding: ItemRecyclerBinding) : RecyclerView.ViewHolder(binding.root){
    fun setBoard(board :Board){
        binding.textNo.text = "${board.no}"
        binding.textTitle.text = "${board.title}"
        var sdf = SimpleDateFormat("yyyy/MM/dd")
        var formattedDate = sdf.format(board.timestamp)
        binding.textDate.text = formattedDate

    }
}
```

### 어댑터 사용해서 연결하기

여태까지 만든 어댑터를 layout과 연결해주면 끝이다....

```Kotlin
  class MainPage : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {

        var binding = ActivityMainPageBinding.inflate(layoutInflater)

        super.onCreate(savedInstanceState)
        setContentView(binding.root)

        //loadData()는 설명하지 않았지만 더미데이터를 생성하는 코드이다.
        val data:MutableList<Board> = lodaData()

        var adapter = CustomAdapter()
        //어댑터안에서 사용하는 MutableList에 data를 넣는다.
        adapter.listData = data

        //어댑터를 연결해준다.
        binding.recycler.adapter = adapter
        //리사이클러뷰를 화면에 보여주는 형태를 결정하는 레이아웃매니저를 연결한다.
        binding.recycler.layoutManager = LinearLayoutManager(this)




    }

```
