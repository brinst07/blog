---
title: "viewpage"
date: 2021-01-17T23:51:11+09:00
author: "brinst"
draft: false
---

## viewPager란?

![image](https://user-images.githubusercontent.com/60083557/104843909-6d1e8e80-5910-11eb-98a8-e4b3415b8625.png)<br>
배달의 민족이나 어플들을 사용하면 상단의 그림들이 스와이프 하면 넘어가거나 한것을 볼수 있었을 것이다. 그것을 구현하기 위한 것이 바로 viewPager이다.

## 구현방법

### 액티비티에 viewPager 만들기

아래 코드 처럼 viewPager를 xml에 추가한다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.viewpager.widget.ViewPager
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="220dp"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### layout 만들기

listView나 viewpager나 그 안에 들어가는 layout아이템을 만들어서 adapter에 넣는 방식으로 진행이 되는 것같다.
아직 안드로이드 초보라서 모르겠지만 지금까지 파악한 개념으로는 해당액티비에 pager든 view를 만들어서 어댑터를 만들고 그안에 dto나 데이터를 넣어 kt파일에서 결합하는 방식으로 진행이 되는거같다.
하단처럼 진행을 했고 맨처음에 constaintLayout으로 만들어지지만 LinearLayout으로 변경하여 만들었다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="220dp"
        android:id="@+id/imageview"
        />
</LinearLayout>
```

### adapter만들기

다음은 어댑터를 만든다.

```kt
package com.brinst.comin

import android.content.Context
import android.view.LayoutInflater
import android.view.View
import android.widget.ImageView
import androidx.viewpager.widget.PagerAdapter
import androidx.viewpager.widget.ViewPager

class ViewPagerAdapter(private val context: Context) : PagerAdapter(){

    private var layoutInflater : LayoutInflater? = null

    val Image = arrayOf(
        R.drawable.ai,
        R.drawable.css,
        R.drawable.html
    )

    override fun getCount(): Int {
        return Image.size
    }

    override fun isViewFromObject(view: View, `object`: Any): Boolean {
        return view === `object`
    }

    override fun instantiateItem(container: View, position: Int): Any {

        layoutInflater = context.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
        val v = layoutInflater!!.inflate(R.layout.viewpager_activity,null)
        val image = v.findViewById<View>(R.id.imageview) as ImageView

        image.setImageResource(Image[position])

        val vp = container as ViewPager
        vp.addView(v,0)

        return v

    }

    override fun destroyItem(container: View, position: Int, `object`: Any) {

        val vp = container as ViewPager
        val v = `object` as View
        vp.removeView(v)
    }
}
```

위와 같은 코드로 구성이 되는데 메소드의 대한 상세설명은 다음과 같다.
|함수|설명|
|--|--|
|instantiateItem(ViewGroup view, int position)|데이터 리스트에서 인자로 넘어온 position에 해당하는 아이템 항목에 대한 페이지를 생성합니다. |
|destroyItem(ViewGroup parent, int position, Object object)|Adapter가 관리하는 데이터 리스트에서 인자로 넘어온 position에 해당하는 데이터 항목을 생성된 페이지를 제거|
|int getCount()|Adapter가 관리하는 데이터 리스트의 총 개수|
|isViewFromObject(View view, Object object) |페이지가 특정 키와 연관되는지 체크|

### MainAcitivy에서 구현

```Kotlin
package com.brinst.comin

import android.content.Intent
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.viewpager.widget.ViewPager
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

    internal lateinit var viewpager : ViewPager

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //viewpager
        viewpager = findViewById(R.id.viewPager) as ViewPager

        val adapter = ViewPagerAdapter(this)
        viewpager.adapter=adapter
        //end viewpager
    }
}
```
