---
title: "default와 new() 제약조건 사용하기"
date: 2017-08-14T15:50:00+09:00
tags : ["csharp"]
categories : ["develop"]

aliases:
    - /develop/2017/08/14/using-default-and-new-keyword.html
---

최근 C# 코드를 작성할 일이 많아졌습니다. 가끔 generic 함수를 작성하곤 하는데, 기본 값을 생성해줘야 할 때가 있었습니다.

이 때 사용하게 되는 키워드가 new와 default입니다. 이 글에서는 class, struct 그리고 enum을 사용할 때 어떤 점을 주의해야 하는지 살펴보도록 하겠습니다.

## new와 default
new는 generic 함수의 제약 조건입니다.
해당 제약 조건을 걸면 generic 함수 내에서 `new T()`와 같은 코드를 사용할 수 있습니다.
이 제약 조건을 걸었을 경우 struct와 enum은 항상 사용가능하고, class는 구현에 따라 달라지게 됩니다.

default는 C#의 기본 키워드로 제약 조건이 필요하지도 않고, 다른 많은 곳에서도 사용할 수 있는 기능입니다.
default를 각 타입에 사용할 경우 아래 표와 같은 값이 생성됩니다.

| type | default value |
| --- | --- |
| class | null |
| struct | Struct { fields = default(type)... } |
| enum | 0 (가장 처음 나오는 0인 item) |

이제 각 타입의 기본값을 설정할 때 주의할 점을 살펴보도록 하겠습니다.

## class
class는 참조 타입(reference type)이기 때문에 null을 허용합니다.
그래서 default를 사용했을 때 값은 항상 **null**이 됩니다.

new의 경우 생성자를 하나도 만들지 않은 상태라면, 항상 사용이 가능합니다(기본 생성자).
반대로 매개 변수가 있는 생성자만 있을 때는 사용할 수 없습니다.

{{< highlight csharp >}}
public class ClassDefault1 {
    public int item1 { get; set; }
    public string item2;
}

public class ClassDefault2 {
    public int item1 { get; set; }
    public string item2;

    public ClassDefault(int item1Value) {
        item1 = item1Value;
    }
}

public TRefType GetDefault<TRefType>()
    where TRefType : new()
{
    return new TRefType();
}

var cls = Getdefault<ClassDefault1>();
// ClassDefault1 { item1 = 0, item2 = null }
var cls2 = GetDefault<ClassDefault2>();
// Error CS0310
{{< / highlight >}}

## struct
struct는 값 타입(value type)입니다.
참조가 아니기 때문에 null을 허용하지 않습니다.

default의 값은 new 구문으로 생성한 값과 같습니다.

{{< highlight csharp >}}
public struct StructDefault {
    public int item1 { get; set; }
    public string item2;

    public StructDefault(int i) {
        item1 = i;
        item2 = "";
    }
}

var str = default(StructDefault);
var str2 = new StructDefault());
// str, str2 == StructDefault { item1 = 0, item2 = null }
{{< / highlight >}}

만약 default나 new로 생성하고자 한다면, 각 멤버에 기본 값을 설정해두면 됩니다.

{{< highlight csharp >}}
public struct StructDefault {
    public int item1 { get; set; } = 10;
    public string item2 = "test";
}

var str = default(StructDefault);
// StructDefault { item1 = 10, item2 = "test" }
{{< / highlight >}}

## enum
enum은 struct와 같은 값 타입입니다.
default와 new를 사용할 때 항상 ['(E)0'이](https://stackoverflow.com/a/4967673) 설정됩니다.

{{< highlight csharp >}}
public enum EnumDefault1 {
    Item1,
    Item2,
    Item3,
    Item4,
}
public enum EnumDefault2 {
    Item1, // 0
    Item2, // 1
    Item3 = 0,
    Item4, // 1
}
public enum EnumDefault3 {
    Item1 = 1,
    Item2, // 2
    Item3, // 3
    Item4, // 4
}

var enm1 = default(EnumDefault1);
// EnumDefault1.Item1
var enm2 = default(EnumDefault2);
// EnumDefault2.Item1
var enm3 = default(EnumDefault3);
// 0

var newEnm = new EnumDefault1();
// EnumDefault1.Item1
{{< / highlight >}}

EnumDefault3은 항목 중 0인 값이 없기 때문에 상수 0이 들어가게 됩니다.
이런 경우가 생길 수 있기 때문에 [MSDN에서는](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/keywords/enum) 상수 0인 값이 항상 enum에 포함되는 것을 권장합니다.

만약, enum만 사용가능한 generic 함수를 만들고 싶다면 [아래와 같은 코드](https://stackoverflow.com/a/79903)를 작성하면 됩니다.

{{< highlight csharp >}}
public TEnum EnumDefault<TEnum>()
    where TEnum : struct, IConvertible
{
    if (!typeof(TEnum).IsEnum) 
   {
      throw new ArgumentException("TEnum must be an enumerated type");
   }
    return default(TEnum);
}
{{< / highlight >}}

다만 이런 형태의 코드를 사용할 경우, 컴파일러 오류로 체크되지 않을 수 있습니다.


## 결론
위 글은 다음과 같이 정리할 수 있습니다.

1. new() 제약 조건: 만약 제약 조건을 설정하였다면, new를 사용하는 것이 더 좋습니다.
2. 제약 조건이 없을 때: class, struct 그리고 enum을 모두 허용하고, `new()` 제약 조건이 없는 경우라면, default를 사용해야 합니다. 하지만, 사용할 때 null 처리를 주의깊게 해야합니다.
3. class: new 제약 조건이 있는 generic 함수를 사용한다면, 매개 변수가 없는 생성자를 구현해 놓는 것이 좋습니다. 기본 생성자를 사용한다면, 각 멤버에 기본 값을 설정해두는 것이 좋습니다.
4. struct: 각 멤버에 기본 값을 설정해두는 것이 좋습니다.
5. enum: 0인 항목이 반드시 1개는 존재하도록 구현하는 것이 좋습니다.

*위 코드는 모두 .net framework 4.0, visual studio 2015 community에서 작성되었습니다.