# ViewPager2

> 코드, 사용법 이런건 웬만하면 안넣음. 모든 글은 내가 생각해보고 탐구해본 것 위주로.
> 코드를 다 적으면 정리하다가 시간이 너무 흘러가서 못하겠다.

## ViewPager란?

* 정의로는 페이지간 전환을 위한 스와이프 동작을 위해 사용하고, 화면 슬라이드 애니메이션등을 지원함.
  앱 쓰다보면 요즘 앱들 많이 쓰는 것 같아 보인다. 1 -> 2 버전 업데이트 하며 세로 페이징이나 오른쪽에서 왼쪽 페이징도 지원한다고 한다.

  ### ViewPager 간단한 사용법

* 기본형(난 이것만 알았다)

```kotlin
  // 3개의 화면을 구성한다고 가정 
  // OneFragment, TwoFragment, ThreeFragment
  
  class ViewPageAdapter(fragmentActivity: FragmentActivity): FragmentStateAdapter(fragmentActivity) {

    // 페이지 갯수 설정
    override fun getItemCount(): Int = 3

    // 불러올 Fragment 정의
    override fun createFragment(position: Int): Fragment {
        return when (position) {
            0 -> HomeFragment()
            1 -> SecondFragment()
            2 -> ThirdFragment()
            else -> throw IllegalArgumentException("Invalid position $position")
        }
    }

    // MainActivity
    class MainActivity : AppCompatActivity() {

      private lateinit var binding : ActivityMainBinding
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          binding = ActivityMainBinding.inflate(layoutInflater)
          setContentView(binding.root)
  
          binding.viewpage.adapter = ViewPageAdapter(this)
      }
  
    }
```  
#

* 동적으로 사용
```kotlin
// 3개의 화면을 구성한다고 가정
// OneFragment, TwoFragment, ThreeFragment
class ViewPageAdapter2(fragmentActivity: FragmentActivity): FragmentStateAdapter(fragmentActivity) {

    var fragments : ArrayList<Fragment> = ArrayList()

    // 페이지 갯수 설정
    override fun getItemCount(): Int = fragments.size

    // 불러올 Fragment 정의
    override fun createFragment(position: Int): Fragment {
        return fragments[position]
    }

    fun addFragment(fragment: Fragment) {
        fragments.add(fragment)
        notifyItemInserted(fragments.size-1)
    }

    fun removeFragment() {
        fragments.removeLast()
        notifyItemRemoved(fragments.size)
    }
}

// MainActivity
class MainActivity : AppCompatActivity() {

    private lateinit var binding : ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val adapter = ViewPageAdapter2(this)
        adapter.addFragment(HomeFragment())
        adapter.addFragment(SecondFragment())
        adapter.addFragment(ThirdFragment())
        binding.viewpage.adapter = adapter

    }

}
```

#

* 같은 Layout에 일부 View만 교체하는 경우 RecyclerViewAdapter 사용
```kotlin
class ViewpageAdapter(val datas : Array<Int>) : RecyclerView.Adapter<ViewpageAdapter.ViewHolder>() {

    inner class ViewHolder(private val binding : ViewpagePostBinding) : RecyclerView.ViewHolder(binding.root){

        fun bind(item : Int){
            binding.ivImage.setImageResource(item)
        }
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(datas[position])
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val binding = ViewpagePostBinding.inflate(LayoutInflater.from(parent.context),parent,false)
        return ViewHolder(binding)
    }

    override fun getItemCount(): Int {
        return datas.size
    }

}

// MainActivity
class MainActivity : AppCompatActivity() {

    private lateinit var binding : ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        val adapter = ViewpageAdapter(item.imgList)
				binding.viewpager.adapter = adapter
    }

}

```
#

### FragmentPagerAdapter vs FragmentStatePagerAdapter

* FragmentPagerAdapter : 모든 프래그먼트를 메모리에 저장해두기 때문에 빠르게 스와이핑해도 메모리에 로드된 프래그먼트를 가져다 쓰므로 버벅이지 않고 화면이 잘 넘어간다.  
* FragmentStatePagerAdapter : 필요에 따라 메모리에서 Fragment를 제거하고 다시 생성하며 상태를 유지한다. 메모리에는 각 프래그먼트의 상태값만 계속 저장하기 때문에 메모리 입장에서는 부담이 덜하다.




#



위처럼 보통 사용하는데 그러면 어느 때 쓰이냐?  
공식 문서에도 TabLayout과 같이 쓰는 경우가 많다고 나와있지만 내가 여러 앱을 사용하면서 본 결과 거의 다 TabLayout이랑 연계해서 사용하였음. 생각해보면 뷰페이저 사용 이유가 화면 스와이프로 넘기는게 대부분이니까 '스와이프해서 화면 넘어가요~' 라고 유저 경험을 가장 향상시켜줄 수 있는게 탭 레이아웃이라 그런거라고 생각한다.
그 외에도 내가 어느 카테고리를 보고있는지 가장 잘 나타내기 쉬운게 TabLayout이기도 하고. 말로는 설명이 어려워도 막상 구현해보면 당연하다고 느껴지는게 이 화면 구성이다.

