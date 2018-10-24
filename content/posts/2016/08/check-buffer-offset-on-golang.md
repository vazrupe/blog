---
title: "Go에서 io.Seeker 인터페이스로 버퍼 오프셋(offset) 확인하기"
date: 2016-08-05T18:48:00+09:00
tags : ["go"]
categories : ["develop"]

aliasies:
    - /develop/2016/08/05/check-buffer-offset-on-golang.html
---
데이터를 읽다보면 현재 오프셋(offset) 위치를 확인해야 할 때가 있기 마련이다.
하지만 Go 언어는 다양한 인터페이스를 통해 값을 읽어오기 때문에 내부에 있는 오프셋을 확인하기 어렵다.
이때는 io.Seeker 인터페이스에 구현된 Seek 함수를 통해 현재 오프셋을 확인가능하다.

{{< highlight go >}}
import (
    "fmt"
    "os"
)

func main() {
    f, err := os.Open("SampleFile....")
    if err != nil {
        panic(err)
    }
    d := make([]byte, 16)
    _, err = f.Read(d)
    if err != nil {
        panic(err)
    }

    offset, err := f.Seek(0, 1)
    if err != nil {
        panic(err)
    }

    fmt.Println(offset) // output: 16
}
{{< / highlight >}}