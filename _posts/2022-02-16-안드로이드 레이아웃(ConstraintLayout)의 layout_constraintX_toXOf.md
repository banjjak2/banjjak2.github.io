---
title: "안드로이드 레이아웃(ConstraintLayout)의 layout_constraintX_toXOf" 
date: 2022-02-16 23:20:16 +0900
categories: [Android]
tags: [Android]
---

# 안드로이드 레이아웃(ConstraintLayout)의 layout_constraintX_toXOf

- layout을 구성할 때 아래와 같은 코드들을 볼 수 있다.

    ```xml
    app:layout_constraintStart_toStartOf="parent"
    ```

- `app:layout_constraintLeft_toLeftOf="parent"`와 같은 속성도 있는데 굳이 Start를 쓰는 이유는 왼쪽에서 오른쪽(LTR)로 작성되는 영어, 한글은 시작 가장자리는 왼쪽이다.

- 그러나 아랍어와 같은 언어들은 오른쪽에서 왼쪽(RTL)로 작성되므로 시작 가장자리가 오른쪽이다.

- 따라서, LTR이든 RTL이든 동일하게 작동할 수 있도록 하기 위함이다.

- 이러한 이유 때문에 제약 조건을 start, end로 주는 것이다.

- 제약 조건의 이름의 구성은 `layout_constraint<Source>_to<Target>Of`로 구성된다.

    ```xml
    <TextView
        android:id="@+id/textview_email"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:text=""
        android:textColor="#CFCFCE"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textview_desc" />

    <View
        android:id="@+id/view_line"
        android:layout_width="0dp"
        android:layout_height="1dp"
        android:layout_marginTop="10dp"
        android:background="#D4D4D3"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/textview_email" />
    ```

- 위와 같은 코드에서 아래 `<View>` 객체의 마지막 줄 `app:layout_constraintTop_toBottomOf="@+id/textview_email"`이 뜻하는 것은 `<View>`의 위쪽(Top)을 `@+id/textview_email`의 아래쪽(Bottom)에 제약 조건을 걸겠다는 뜻이 된다.

