---
title: "Trunk Based Development - một Git workflow giúp giảm cơn đau đầu resolve conflict"
description: "Ở công ty cũ của mình (tạm gọi là công ty A), source code của dự án siêu to khổng lồ, mới clone về đã 40GB, compile và build các kiểu thì còn lên đến gần 100GB."
date: 2023-05-03T02:07:00+07:00
image: tbd.jpeg
categories: Technical
tags: [Git]
---

## Case study
### Câu chuyện đầu tiên
Ở công ty cũ của mình (tạm gọi là công ty A), source code của dự án siêu to khổng lồ, mới clone về đã 40GB, compile và build các kiểu thì còn lên đến gần 100GB. Mỗi lần anh em code một feature nào đó, thường sẽ checkout ra một branch mới là feature_x. Feature nhỏ thì không sao, chứ feature lớn mà thay đổi vài chục hay đến cả trăm file, xong merge vào branch chính thì đúng là ác mộng, vì phải resolve conflict từ code của branch feature_a, feature_b nào đó merge trước đó.

### Câu chuyện thứ hai
Ở công ty khác của mình (công ty B), do đặc thù của dự án mà project được chia thành 5 team nhỏ, mỗi team tầm 3 developer. Mỗi team nhỏ sẽ phụ trách một vài feature trong 1 sprint. Khi bắt đầu sprint, dev lead tạo branch cho từng team, và sẽ merge 5 branch đó sau khi kết thúc sprint. Vấn đề phát sinh tương tự là merge code xảy ra khá nhiều conflict, và các team lại mất thời gian test lại feature của mình để đảm bảo không xảy ra bug sau khi merge.

## Vậy Trunk Based Development là gì?
Nói một cách ngắn gọn, **Trunk Based Development** (từ sau mình sẽ viết tắt **TBD**) là một **source-control branching model** (mô hình làm việc với các nhánh) mà tất cả developers sẽ làm việc trên **một branch duy nhất**, gọi là **trunk** (nghĩa là cái thân cây), tránh việc tạo ra các feature branch siêu to khổng lồ. **Trunk** branch cần đảm bảo rằng nó có thể sẵn sàng release bất cứ thời điểm nào. Trong các dự án, mọi người thường sẽ đặt tên cho trunk branch là **master** hoặc **dev**.

TBD lại chia thành 2 mô hình nhỏ hơn, phù hợp với từng team. Chúng ta sẽ cùng đi tìm hiểu ngay sau đây.
### TBD cho team nhỏ
![Small TBD](https://images.viblo.asia/3c277b1e-cfc8-4b33-964f-240189208025.png)
Trong mô hình này, toàn bộ dev team sẽ push code trực tiếp lên trunk branch. Tuy nhiên, mô hình này mình thấy có rủi ro rất lớn về chất lượng code, vì code được push trực tiếp mà không qua review. Vậy nên, để áp dụng mô hình này, các dev tham gia project cũng phải có technical skill rất tốt, để đảm bảo rằng từng dòng code mình push lên không gây bug cho tất cả anh em.

Cá nhân mình đánh giá mô hình này có lẽ chỉ phù hợp với team size từ 5 người trở xuống.

### TBD cho team to
![Scaled TBD](https://images.viblo.asia/8cfdf280-8b7a-4751-b50e-4706b9753669.png)

Với TBD cho một team lớn, các feature branch vẫn được tạo, nhưng điểm khác biệt là nó chỉ được tạo trong một thời gian ngắn, chỉ vài commit là merge lại trunk branch ngay.

Ví dụ khi được giao task code Onboarding feature, với Git flow mà thông thường mọi người đang áp dụng, các bạn sẽ làm những bước như sau:
1. Tạo **feature/onboarding** branch
2. Code Welcome screen rồi commit
3. Code Register screen rồi commit tiếp
4. Code Login screen rồi commit nữa
5. Code xong hết rồi thì tạo merge request, chờ dev khác review code
6. Review xong thì merge **feature/onboarding** branch vào **dev** branch

Với quy trình như trên, merge request có thể chứa vài chục file thay đổi, và review đống code ấy thực sự là ác mộng. Đôi khi, vào thời điểm chúng ta tạo merge request, còn xảy ra conflict với code của một feature nào đó đã merge vào **dev** branch trước. Và chúng ta lại phải hì hục ngồi resolve conflict. 

![](https://images.viblo.asia/766696b2-7172-42b3-b0d8-2e1af375f59d.png)

Đối với reviewer, đôi khi cách để thoát khỏi ác mộng là không mơ nữa, nghĩa là bác senior dev sẽ nhanh chóng để lại dòng comment ngắn gọn **LGTM** (Look good to me), vậy là anh chàng junior dev dễ dàng merge code vào **dev** branch. Hoặc mặc dù bác senior dev đã cố gắng review cẩn thận, nhưng với lượng code thay đổi lớn như vậy thì vẫn bị lọt một vài bug. Tựu chung lại thì quy trình này vẫn có rủi ro trong việc để lọt bug.

Đối với TBD, quy trình sẽ điều chỉnh lại một chút như sau:
1. Tạo **feature/welcome_screen** branch
2. Code Welcome screen rồi commit, tạo merge request luôn
3. Trong lúc chờ review thì code Register screen tiếp
4. Khi Merge request của Welcome screen được approve, lại checkout ra **feature/register_screen** branch để tạo merge request mới
5. Tương tự như vậy với Login screen

Giờ đây, mỗi merge request của chúng ta chỉ chứa vài file thay đổi, bác senior dev có thể review code dễ dàng hơn.

Và kể cả khi branch release có bug, chúng ta cần hotfix thì dev cũng không merge trực tiếp vào release branch như các mô hình khác, mà tất cả đều phải merge vào trunk branch.

### Feature flag
Các bạn có thể thắc mắc rằng làm như vậy thì trên **dev** branch sẽ chứa đầy code dở dang của các feature. Ví dụ app sẽ hiển thị Welcome screen với UI mới, trong khi Register và Login screen thì vẫn là UI cũ. Như vậy, làm sao có thể đảm bảo rằng trunk branch có thể release bất cứ lúc nào?

Chìa khoá để giải quyết vấn đề này chính là **Feature flag**. Với mỗi feature dang dở như vậy, bạn cần đặt cho nó một cái flag, để khi bật flag, tất cả code của feature mới sẽ hoạt động, còn nếu tắt flag, app sẽ hoạt động như code hiện tại. Ví dụ:
```java
if (ONBOARDING_FLAG == FLAG_ON) {
    displayNewWelcomeScreen()
} else {
    displayOldWelcomeScreen()
}

if (ONBOARDING_FLAG == FLAG_ON) {
    displayNewRegisterScreen()
} else {
    displayOldRegisterScreen()
}

if (ONBOARDING_FLAG == FLAG_ON) {
    displayNewLoginScreen()
} else {
    displayOldLoginScreen()
}
```

Đôi khi, bạn muốn revert một feature thì cũng chỉ cần tắt flag của nó đi là xong. Quá nhanh chóng phải không nào?
## Ưu điểm
### Giảm conflict, giảm thời gian review code
Như mình đã trình bày ở trên, TBD yêu cầu mọi người phải nhanh chóng merge code của mình vào trunk branch, điều này giúp giảm conflict code, đồng thời giảm thời gian review code.

Với case study thứ 2 của công ty B, sau một thời gian thấy mô hình hiện tại không hiệu quả, mình đã đề xuất với dev lead áp dụng TBD. Và đúng như mong đợi, với mỗi sprint, hầu như team mình không còn phải dành quá nhiều thời gian resolve conflict nữa.
### CI/CD
TBD thực sự rất hữu ích cho CI/CD. Nghĩ mà xem, giờ đây bạn chỉ cần setup và run CI/CD trên một branch duy nhất. Tất cả những commit của mọi người sẽ nhanh chóng được check coding convention, unit test cẩn thận, giúp cả team dễ dàng phát hiện ra lỗi sai và fix nó kịp thời.
### A/B Testing
Với feature flag, bạn cũng dễ dàng bật tắt các feature, giúp Product Manager thuận tiện setup các A/B Testing. Từ đó có thể đo lường để đưa ra đánh giá chính xác về hiệu quả của một feature mới.

Ở case study thứ nhất của công ty A, thực sự thì chúng mình đã apply feature flag cho rất nhiều feature quan trọng để thực hiện A/B testing.
### Nhanh chóng deliver sản phẩm
Vì code ở trunk branch luôn sẵn sàng release nên mình thấy nó khá phù hợp với các start-up. Bất cứ khi nào bạn muốn release một version mới, chỉ cần checkout từ trunk branch, bật các feature flag đã hoàn thành và tắt các feature flag còn dang dở, rồi build code.
## Nhược điểm
### Quá nhiều code dư thừa
Nói đi cũng phải nói lại, việc áp dụng **feature flag** và if else quá nhiều có thể khiến code của bạn trở nên phức tạp. Đôi khi có một số function, code if else không clear có thể khiến reviewer còn cảm thấy khó hiểu hơn.

Và khi một feature được release, bạn cũng lại phải quay lại để xoá code flow cũ đi, như các function `displayOldWelcomeScreen`, `displayOldRegisterScreen` và `displayOldLoginScreen` trong ví dụ trên.
### Không phù hợp với team có nhiều junior
Việc tạo merge request liên tục yêu cầu các dev phải cẩn trọng, đảm bảo rằng mỗi commit của mình đã pass tất cả coding convention check hay unit test ở local, và đặc biệt là không có critical bug. Vì nếu có bug trên trunk branch khiến cho không thể mở được app chẳng hạn, nó sẽ block công việc của tất cả các dev khác.

Nếu dùng feature flag, dev còn cần test cả 2 flow cũ và mới để đảm bảo cả 2 đều hoạt động đúng.

Vậy nên mình nghĩ rằng một team gồm chủ yếu là senior sẽ phù hợp hơn với mô hình này.
## Lời kết
TBD là một mô hình làm việc mà mình đã áp dụng ở một số công ty và thấy nó hoạt động khá hiệu quả với tình hình của team mình lúc đó. Các bạn có thể nghiên cứu ưu, nhược điểm của mô hình so với team mình và áp dụng thử nhé.
## Reference
* https://trunkbaseddevelopment.com/
