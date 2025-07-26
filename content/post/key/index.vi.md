---
title: "Chuyên án điều tra: Giải mã bí ẩn Key trong Flutter"
description: "Key xuất hiện tràn ngập trong Flutter, từ StatefulWidget tới StatelessWidget. Vậy nhưng tưởng như rất gần mà ngờ đâu đã quá xa, nghĩ rằng đã trở nên thân thuộc mà lại vô cùng bí ẩn. Hôm nay, đội cảnh sát hình sự Flutter Việt Nam sẽ đưa anh em đi sâu vào ngõ ngách của băng đảng Flutter, giải mã vai trò của Key."
date: 2025-05-12T00:00:00+07:00
slug: key
image: key.webp
toc: true
categories: [Technical, Flutter]
tags: [Flutter, Key]
---

> Đã quá nhàm chán với những bài viết học thuật? Vậy thì hôm nay mình sẽ mang đến một bài viết với giọng văn hình sự, mời anh em giải trí.

# Mở đầu chuyên án

**Key** xuất hiện tràn ngập trong Flutter, từ **StatefulWidget** tới **StatelessWidget**. Vậy nhưng tưởng như rất gần mà ngờ đâu đã quá xa, nghĩ rằng đã trở nên thân thuộc mà lại vô cùng bí ẩn. Tuy anh em Flutter developer thường xuyên làm việc với **Widget**, nhưng **Key** - thứ luôn âm thầm đứng phía sau các vụ *chuyển giao quyền lực* - lại hiếm khi được chú ý.

Hôm nay, đội cảnh sát hình sự Flutter Việt Nam sẽ đưa anh em đi sâu vào ngõ ngách của băng đảng **Flutter**, giải mã vai trò của **Key** trong việc **tối ưu hóa hiệu năng** ứng dụng Flutter, đồng thời khám phá các **best practice** để sử dụng chúng.

# Đi tìm ông trùm đứng sau

Định nghĩa của Key trong [document](https://api.flutter.dev/flutter/foundation/Key-class.html) nói rằng:

> A Key is an identifier for Widgets, Elements and SemanticsNodes.
>
> A new widget will only be used to update an existing element if its key is the same as the key of the current widget associated with the element.
>
> *Tạm dịch:*
>
> *Key là mã định danh cho Widget, Element và SemanticsNodes.*
>
> *Một widget mới chỉ được sử dụng để cập nhật một element đã tồn tại nếu key của nó giống với key của widget hiện tại được liên kết với element đó.*

Như anh em đã biết thì trong Flutter, mọi thứ đều là **Widget**. Băng đảng Widget này gồm nhiều thành phần đã quen mặt với anh em như **Row**, **Column**, **Container**... Thế nhưng lũ Widget này chỉ là tay chân lâu la, theo các thông tin tình báo chúng tôi có được, băng đảng này được điều hành bởi một ông trùm khét tiếng: **Element**.

Hắn là kẻ thao túng tất cả Widget, từ việc gọi hàm `initState`, `build`, `dispose` của Widget đến vai trò quản lý Widget Tree. **Element** đồng thời là một mắt xích quan trọng kết nối giữa **Widget** và **RenderObject** - kẻ giúp vẽ UI lên màn hình – để điều chế ma... à nhầm, để tạo nên những tác phẩm nghệ thuật tinh xảo.

Tuy nhiên, hôm nay chúng ta không bắt ôm trùm hay triệt phá toàn bộ băng đảng này, chỉ cần đơn giản nhớ rằng **Element** là kẻ đứng sau điều phối tất cả. Cá nuôi chưa lớn thì chưa nên cất vó. Mục tiêu của chuyên án là điều tra **Key** và 4 gã **tứ đại cao thủ**.

# Key

Khi bạn rebuild ứng dụng Flutter, bạn có biết chuyện gì xảy ra trong bóng tối không? **Element** quyết định những **Widget** nào sẽ được giữ lại, thay thế, hay hủy bỏ. Và đây là nơi mà **Key** bắt đầu thể hiện quyền lực.

Trong giới Widget, mỗi khi thay đổi diễn ra, Widget không chỉ đơn giản là **update** mà là bị **huỷ bỏ** rồi **tái sinh**. Key chính là *căn cước công dân* để các Widget giữ nguyên danh tính của mình khi tái sinh. Sau khi Widget tree bị rebuild, Element sẽ dựa vào **Widget type** và **Key** để quyết định xem Element có bị rebuild hay không. Trước mắt, nếu Widget type khác nhau, chắc chắn Element sẽ bị huỷ bỏ đi tạo lại. Mà bạn biết đấy, việc rebuild cả ông trùm thì sẽ rất tốn kém so với việc rebuild đám lâu la Widget. Việc này gây ra những vấn đề không mong muốn về performance và đôi khi làm ứng dụng của bạn lag.

Còn nếu Widget type giống nhau, chúng sẽ tiếp tục so sánh đến Key, nếu Key giống nhau thì Element chỉ update widget. Ngược lại, Element sẽ bị deactivate, nghĩa là tạm gỡ ra khỏi Element tree và có khả năng được gắn lại vào tree sau.

Key gồm 2 loại chính là **LocalKey** và **GlobalKey**, trong đó LocalKey lại được chia ra thành **UniqueKey**, **ValueKey** và **ObjectKey**. Sau đây, chúng ta sẽ đi bóc trần từng gã.

# UniqueKey - Sát thủ không thể truy dấu

Một kẻ bí ẩn, thoắt ẩn thoắt hiện, không bao giờ xuất hiện hai lần với cùng một giá trị. Hắn tạo ra **các giá trị duy nhất** để giúp Flutter **phân biệt hai Widget** dù chúng có cùng type. Chúng ta cần tới hắn khi **không muốn tái sử dụng** bất kỳ Widget nào, đảm bảo rằng Widget được rebuild hoàn toàn.

Đầu tiên, chúng ta tạo một Item widget chỉ đơn giản là hiển thị một Text, nhưng mình sẽ dùng `StatefulWidget` để các bạn thấy rõ được quá trình nó được init và rebuild.

```dart
class Item extends StatefulWidget {
  final String text;

  const Item({super.key, required this.text});

  @override
  State<Item> createState() => _ItemState();
}

class _ItemState extends State<Item> {
  @override
  void initState() {
    super.initState();
    debugPrint('[_ItemState.initState] key = ${widget.key}, text = ${widget.text}');
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('[_ItemState.build] key = ${widget.key}, text = ${widget.text}');
    return Text(widget.text);
  }
}
```

Tiếp theo, mình tạo một `ListView` chứa các Item widget đó, bạn có thể thấy mình chưa truyền Key gì cho các item.

```dart
final names = ['Henry', 'Techie', 'Nam', 'Anh', 'Nguyen'];

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Demo Key')),
      body: ListView.builder(
        itemCount: _counter,
        itemBuilder: (context, index) => Item(
          text: names[index],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        child: Icon(Icons.add),
      ),
    );
  }
}
```

Chúng ta có thể xem log và thấy khi bạn click `FloatingActionButton` để tạo lần lượt từng item, chỉ widget mới nhất được tạo là gọi đến `initState`, còn những widget khác chỉ `rebuild`. Đây là log khi mình tạo item thứ 5:

```
[_ItemState.build] key = null, text = Henry
[_ItemState.build] key = null, text = Techie
[_ItemState.build] key = null, text = Nam
[_ItemState.build] key = null, text = Anh
[_ItemState.initState] key = null, text = Nguyen
[_ItemState.build] key = null, text = Nguyen
```

Bây giờ, đã đến lúc giao việc cho gã sát thủ UniqueKey. Mình sẽ thêm UniqueKey vào Item như sau:

```dart
body: ListView.builder(
  itemCount: _counter,
  itemBuilder: (context, index) => Item(
    key: UniqueKey(),
    text: names[index],
  ),
),
```

Mọi chuyện đã trở nên hoàn toàn khác, khi click `FloatingActionButton` để tạo lần lượt từng item, tất cả các widget đều bị tạo lại. Thậm chí giá trị của Key còn thay đổi sau mỗi lần. Đây là log khi mình tạo item thứ 5, bạn có thể thấy không chỉ riêng item thứ 5 mà tất cả item từ 1 đến 4 cũng đều bị init lại:

```
[_ItemState.initState] key = [#c24b5], text = Henry
[_ItemState.build] key = [#c24b5], text = Henry
[_ItemState.initState] key = [#02979], text = Techie
[_ItemState.build] key = [#02979], text = Techie
[_ItemState.initState] key = [#0a0db], text = Nam
[_ItemState.build] key = [#0a0db], text = Nam
[_ItemState.initState] key = [#23d4d], text = Anh
[_ItemState.build] key = [#23d4d], text = Anh
[_ItemState.initState] key = [#614a4], text = Nguyen
[_ItemState.build] key = [#614a4], text = Nguyen
```

# ValueKey - Gã đồ tể đáng tin cậy

Hắn là một kẻ đầu óc đơn giản nhưng làm việc rất hiệu quả, **ValueKey** là lựa chọn hoàn hảo khi bạn có một giá trị định danh rõ ràng cụ thể, chẳng hạn như `String` hoặc `int`. Gã đồ tể này giúp Element biết đâu là Widget cần giữ lại chỉ dựa vào giá trị định danh đó, vậy nên chúng ta có thể **tái sử dụng** Widget khi giá trị Key không thay đổi.

Đầu tiên, chúng ta hãy thử update `ListView` trong ví dụ trên thành `ReorderableListView`. Vì `ReorderableListView` bắt buộc mỗi item của nó đều phải có Key, nên chúng ta sẽ bắt đầu ngay với ví dụ Item đã có `ValueKey`.

```dart
body: ReorderableListView(
  onReorder: (oldIndex, newIndex) {
    setState(() {
      if (newIndex > oldIndex) newIndex -= 1;
      final item = names.removeAt(oldIndex);
      names.insert(newIndex, item);
    });
  },
  children: names
      .map((item) => Item(
          key: ValueKey(item),
          text: item,
        ))
      .toList(),
),
```

Bây giờ mỗi khi kéo thả để thay đổi thứ tự các item, chúng ta sẽ thấy các item được rebuild:

```
[_ItemState.build] key = [<'Henry'>], text = Henry
[_ItemState.build] key = [<'Techie'>], text = Techie
[_ItemState.build] key = [<'Nam'>], text = Nam
[_ItemState.build] key = [<'Anh'>], text = Anh
[_ItemState.build] key = [<'Nguyen'>], text = Nguyen
```

Vậy, nếu chúng ta thử thay đổi giá trị của một item sau khi kéo thả bằng cách update function `onReorder` như sau thì sao? Ví dụ, khi kéo thả item 'Henry', mình sẽ thay đổi giá trị của nó thành 'Henry Changed'.

```dart
onReorder: (oldIndex, newIndex) {
  setState(() {
    if (newIndex > oldIndex) newIndex -= 1;
    final item = '${names.removeAt(oldIndex)} Changed';
    names.insert(newIndex, item);
  });
},
```

Khi đó, giá trị thay đổi dẫn tới ValueKey cũng bị thay đổi, item sẽ bị tạo lại như sau:

```
[_ItemState.initState] key = [<'Henry Changed'>], text = Henry Changed
[_ItemState.build] key = [<'Henry Changed'>], text = Henry Changed
```

# ObjectKey - Quân sư đa mưu túc trí

Ngược lại với Value Key, gã này là một bậc thầy chiến lược, giữ trong tay cả một object phức tạp. **ObjectKey** dựa trên **tham chiếu** đến object. 2 Key chỉ được coi là **giống nhau** nếu chúng tham chiếu đến cùng một object.

Tương tự ví dụ trên về ValueKey, chúng ta sẽ update lại một chút để sử dụng ObjectKey thay cho ValueKey.

```dart
class _HomePageState extends State<HomePage> {
  final List<Person> people = [
    Person(name: "Henry"),
    Person(name: "Techie"),
    Person(name: "Nam"),
    Person(name: "Anh"),
    Person(name: "Nguyen"),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Demo Key')),
      body: ReorderableListView(
        onReorder: (oldIndex, newIndex) {
          setState(() {
            if (newIndex > oldIndex) newIndex -= 1;
            final item = people.removeAt(oldIndex);
            people.insert(newIndex, item);
          });
        },
        children: people
            .map((item) => Item(
                key: ObjectKey(item),
                text: item.name,
            ))
            .toList(),
      ),
    );
  }
}

class Person {
  final String name;
  Person({required this.name});
}
```

# GlobalKey - Người quản gia quyền năng

Trong băng đảng, GlobalKey là kẻ mạnh nhất. Hắn biết tất cả mọi thứ trong Widget Tree. Không chỉ lưu danh tính, hắn còn quản lý toàn bộ trạng thái và cho phép truy cập trực tiếp đến `State`. Điều này mang lại sự linh hoạt nhưng cũng dễ bị lạm dụng.

Ví dụ thường thấy nhất là dùng GlobalKey để kiểm soát và xác nhận trạng thái của `Form`.

```dart
class GlobalKeyExample extends StatelessWidget {
  final GlobalKey<FormState> formKey = GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Form(
      key: formKey,
      child: Column(
        children: [
          TextFormField(validator: (value) => value!.isEmpty ? 'Required' : null),
          ElevatedButton(
            onPressed: () {
              if (formKey.currentState!.validate()) {
                print('Form is valid!');
              }
            },
            child: Text('Submit'),
          ),
        ],
      ),
    );
  }
}
```

# Best Practices khi dùng Key

**Key** không chỉ là một công cụ, mà là *bảo bối* để kiểm soát ứng dụng. Việc hiểu và sử dụng đúng Key không chỉ giúp bạn tối ưu hóa performance mà còn đảm bảo logic của ứng dụng luôn ổn định và chính xác. Lạm dụng nó có thể làm code trở nên phức tạp không cần thiết, vậy nên hãy chỉ sử dụng khi cần. Hãy nhớ rằng trong thế giới Flutter đầy biến động này, Key chính là chiếc chìa khóa cho sự mượt mà của ứng dụng mà anh em đang phát triển!
