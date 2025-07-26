---
title: "Quy trình làm việc chuẩn chỉ với Git"
description: "Trải qua một thời gian đi làm, lang bạt qua vài công ty, mình nhận thấy ở mỗi nơi lại có quy trình làm việc với Git khác nhau. Bài viết này giới thiệu quy trình làm việc với Git mà mình nghĩ là chuẩn chỉ và cũng đang áp dụng ở công ty hiện tại."
date: 2020-11-10T17:19:00+07:00
slug: git-process
image: git.webp
categories: [Technical]
tags: [Git]
---

# Lời mở đầu
Trải qua một thời gian đi làm, lang bạt qua vài công ty, mình nhận thấy ở mỗi nơi lại có quy trình làm việc với Git khác nhau. Bài viết này giới thiệu quy trình làm việc với Git mà mình nghĩ là chuẩn chỉ và cũng đang áp dụng ở công ty hiện tại. Vậy nên, mình sẽ không giới thiệu hết các lệnh Git, mà chỉ lướt qua những lệnh mình nghĩ đủ dùng cho quá trình làm việc của các bạn ở công ty.

# Quy trình
## Ngày đầu tiên đi làm
Đơn giản quá nhỉ, ngày đầu tiên đi làm thì còn lệnh gì ngoài `git clone` nữa cơ chứ. Khi bạn muốn lấy source của team về, thì chỉ cần mở terminal, gõ:
```shell
git clone <url>
```

Thêm 1 típ nhỏ, là nếu bạn muốn tên folder sau khi clone về khác tên project trên remote thì hãy đặt tên folder ở cuối lệnh:
```shell
git clone <url> folder_name
```

## Những ngày bình thường
### Một mình một ngựa
Sếp giao một feature mới, bắt tay vào làm thôi nào. Khoan đã, nếu bạn đang ở 1 branch khác, đừng quên checkout về branch chính của team (thường branch chính sẽ đặt tên là master):
```shell
git checkout master
```
Rồi pull code mới nhất về:
```shell
git pull
```
Để kiểm tra code ở local đã update chưa, bạn hãy thử kiểm tra bằng `git log`. Mình thì thường dùng lệnh này để xem cho gọn:
```shell
git log --oneline
```
Sau đó, checkout ra một branch mới để bắt đầu làm feature của bạn. Thêm param `-b` sẽ giúp bạn tạo branch mới và checkout sang đó luôn:
```shell
git checkout -b feature_branch
```
...

Code... code... code......

...

Xong rồi, bây giờ thì bắt đầu add những file bạn đã sửa vào stage. Thường thì các IDE hiện tại đều support bạn add và commit nhanh hơn, nhưng nếu bạn vẫn muốn gõ tay thì hãy gõ:
```shell
git add .
```
Lệnh vừa rồi đã giúp bạn add tất cả những file vừa sửa vào stage. Sau đó thì commit:
```shell
git commit -m "Fix all bugs"
```
**Note:** Cách đặt tên branch và commit cũng nên rõ ràng, thể hiện branch đó, commit đó thực hiện feature gì hay là fix bug gì. Điều này hoàn toàn phụ thuộc vào rules của từng team. Có những team đặt tiền tố là ID của task, nhưng có team lại đặt tiền tố là mục đích của commit đó là gì, ví dụ như feature/fixbug/...

Và cuối cùng là push code lên repository:
```shell
git push origin feature_branch
```
Bây giờ thì lên repository, tạo merge request cho sếp review. Trong lúc đó thì tranh thủ đi làm cốc coffee để chờ bước tiếp theo.

### Feature nhiều người làm
Nếu branch của bạn có nhiều người làm chung, bạn chưa kịp push code thì đã có người khác push trước. Vậy thì trước khi push thì hãy pull theo cách này:
```shell
git pull --rebase
```
Commit của bạn sẽ được đẩy lên trên commit của đồng nghiệp trong log.

Trong trường hợp bạn muốn lấy code về nhưng chưa muốn merge thì hãy dùng lệnh:
```shell
git fetch
```
Theo mình thì `pull = fetch + merge`.

### Merge code
Sau một hồi review thì sếp cũng đồng ý cho bạn merge code, nhưng vấn đề là trong quá trình review thì bạn có thêm một vài commit để thêm, sửa, xóa file. Bạn muốn rebase những commit đó lại thành 1, hoặc đơn giản chỉ muốn sửa tên hay xóa commit nào đó. Giả sử bạn có 3 commit cần hợp nhất:
```shell
git rebase -i HEAD~3
```
Terminal hiện ra cho bạn khá nhiều option như edit, reword, squash... Thay chữ pick ở đầu dòng bằng option tương ứng mà bạn muốn dùng. Sau đó bấm Ctrl + O để ghi đè, tiếp tục Ctrl + X để thoát.

Còn một vấn đề nữa là branch master đã có người khác push thêm code. Bạn vẫn có thể merge, nhưng nó sẽ tạo thêm 1 commit merge. Vậy nên mình thường sẽ rebase và merge fast forward.
Đầu tiên, vẫn là:
```shell
git fetch
```
Sau đó là lệnh rebase. Bạn vẫn phải đứng ở `feature_branch` để thực hiện lệnh này:  
```shell
git rebase origin/master
```
Nói một cách dễ hiểu, lệnh rebase sẽ giúp bạn lấy những code mới nhất từ branch master về, sau đó *"viết lại"* branch feature của bạn để đẩy commit của bạn lên trên cùng.
Cuối cùng là push force lên feature branch. Push force sẽ apply toàn bộ log ở local của bạn lên branch ở repo, bất chấp log 2 nơi khác nhau:
```shell
git push -f origin feature_branch
```
Nếu branch một mình bạn làm thì có thể push force thoải mái. Nhưng **hãy cân nhắc cẩn thận khi push force lên branch có nhiều người làm chung**, vì nó sẽ dễ gây conflict cho người khác. Chỉ làm điều này khi bạn đã chắc chắn hoàn thành feature của mình.

Sau đó, merge code ở merge request trên repo. Vậy là bạn đã hoàn thành xuất sắc task đầu tiên rồi.

## Những ngày khủng hoảng
### Reset
Thực ra thì trong quá trình làm có thể bạn có nhầm lẫn gì đó mà cần revert code. Git reset sẽ có 3 option dành cho bạn.

Reset commit nhưng code vẫn ở trong stage, sẵn sàng cho bạn commit lại:
```shell
git reset --soft commit_id
```
Reset commit và đẩy code ra khỏi stage. Bạn cần dùng `git add` trước khi có thể commit lại:
```shell
git reset --mixed commit_id
```
Reset commit và xóa toàn bộ code bạn đã làm: 
```shell
git reset --hard commit_id
```

### Stash 
Bạn có thể dùng cái này như một cứu cánh để lưu tạm code trước khi thực hiện các lệnh rebase hay checkout sang branch khác mà bị conflict. Bạn cứ tưởng tượng nó như một tờ giấy nháp lưu theo cấu trúc stack vậy.

Khi muốn lưu tất cả những thay đổi hiện tại vào stash:
```shell
git stash
```
Khi muốn apply stash cuối cùng vừa lưu:
```shell
git stash pop
```
Hiện tại mình đang dùng chủ yếu Android Studio, và nó có sẵn Shelf với chức năng tương tự nên hiện tại không còn dùng đến git stash.

Bạn có thể tham khảo thêm ở [đây](https://git-scm.com/docs/git-stash).

# Kết
Thực ra cái tiêu đề chỉ giật tít câu view thôi chứ không có gì là chuẩn chỉ cả. Vì quy trình của mỗi công ty là khác nhau, yêu cầu của dự án là khác nhau. Nếu dự án bạn ít người, cần ưu tiên tốc độ làm việc thì có thể bỏ bớt quy trình, push thẳng code lên master. Còn nếu dự án của bạn có nhiều người cùng làm, yêu cầu quy trình khắt khe thì có thể không cho push force lên remote chẳng hạn. Vậy quy trình của công ty của bạn thế nào? Cùng chia sẻ với mình nhé.

Thanks for reading!