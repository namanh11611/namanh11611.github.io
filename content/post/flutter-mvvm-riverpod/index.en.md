---
title: "Flutter MVVM Riverpod Starter: Build Apps Lightning Fast for Vibe Coders"
description: "Recently, I've spent time building apps like Habit Tree and Speakie, or occasionally working on some outsource projects for clients. Every time I start a new project, I spend a whole day setting up architecture, routing, auth, state management... for the app. So, from real needs, I thought about building a lightweight Flutter base project to start as quickly as possible."
date: 2025-05-11T00:00:00+07:00
slug: flutter-mvvm-riverpod
image: fmr.webp
toc: true
categories: [Technical, Flutter]
tags: [Flutter, MVVM, Riverpod]
---

*Photo by [7](https://unsplash.com/@4000km?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/a-silver-and-red-train-traveling-down-train-tracks-u6QssbF_9JM?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)*

Recently, I've spent time building apps like [**Habit Tree**](https://habittree.xyz) and [**Speakie**](https://speakie.xyz) (you can see more at [**Areser**](https://areser.net)), or occasionally working on some outsource projects for clients. Every time I start a new project, I spend a whole day setting up architecture, routing, auth, state management... for the app. So, from real needs, I thought about building a **lightweight Flutter base project** to start as quickly as possible. And that's why **Flutter MVVM Riverpod Starter** was born.

> Check it out here: https://github.com/namanh11611/flutter_mvvm_riverpod

# 🚀 Why choose this starter?

My goal is to create a lightweight template, but powerful enough for **indie hackers** and **solo developers**. That's why I didn't choose **Clean Architecture**—it's too bulky for my target users. Think about it: for projects done solo or with a small team of 2-3 people, **99.99%** of the time you won't need an **Abstract Repository** for multiple implementations. **Andrea**—a well-known Flutter developer—also mentioned this in [Flutter App Architecture: The Repository Pattern](https://codewithandrea.com/articles/flutter-repository-pattern/#repositories-may-not-need-an-abstract-class).

After weighing the options, I chose the **MVVM** (Model - View - ViewModel) architecture, simply using ViewModel to separate logic from UI.

I also chose **Riverpod**, a state management solution for flexible state handling. I know most of you use **BloC** more, but I'm not judging which is better—it's just that **Riverpod** lets me write shorter code than **BloC**.

Next is the Backend. I know most solo developers will integrate **Firebase**, but it has one issue... **EXPENSIVE** (when your project starts to scale). So I chose **Supabase**, similar to **Firebase** but CHEAPER. I've already integrated setup code and some authentication functions.

If you want to monetize your app, you can't miss **RevenueCat**, which helps manage in-app purchases and subscriptions.

There's also Dark/Light Theme, Localization, routing, local storage, analytics, crashlytics... all ready to go.

# 📚 Libraries Used

| Purpose        | Libraries                                                  |
|----------------|------------------------------------------------------------|
| State          | `flutter_riverpod`, `riverpod_annotation`                  |
| Auth & Backend | `supabase_flutter`, `google_sign_in`, `sign_in_with_apple` |
| Navigation     | `go_router`                                                |
| UI/UX          | `google_fonts`, `flutter_svg`, `shimmer`, `lottie`         |
| Storage        | `shared_preferences`, `sqflite`                            |
| HTTP           | `dio`, `connectivity_plus`                                 |
| Utils          | `uuid`, `envied`, `easy_localization`                      |
| Monetization   | `purchases_flutter`, `in_app_purchase`                     |
| Analytics      | `firebase_analytics`, `firebase_crashlytics`               |

# 🏗 Project Architecture

For project architecture, I chose Feature-first (layer folders inside feature folders). For each feature like `authentication`, `onboarding`... I create a folder inside `features`. Then inside that folder, I create layer folders like `model`, `repository`, `ui`.

```bash
lib/
├── constants/         # Constants, global config
├── environment/       # Environment variables
├── extensions/        # Extension/helper methods
├── features/          # Feature modules by folder
│   ├── authentication/
│       ├── model/
│       ├── repository/
│       ├── ui/
│   ├── onboarding/
│   ├── home/
│   ├── profile/
│   ├── premium/
├── routing/           # Route config with go_router
├── theme/             # UI config
└── utils/             # Common utilities
```

You'll notice there are no layers like `domain` or `data`.

Usually with Clean Architecture, your `domain` would have `model`, `abstract repository`, `use case`, but now, without needing `abstract repository` and `use case`, I can simplify it to just `model`.

As for `data`, for features that only need to access remote or local data, I'll call functions directly in the `repository`. For features needing both types of data, I'll consider creating a `data source` if necessary.

# 🎉 Conclusion

The **vibe coding** movement is growing, but I still want to **vibe code with control**. With this base project, I can guide AI to code the way I want. If it goes off track, I can quickly adjust.

As mentioned from the start, this project is for **indie hackers** and **solo developers** who want to launch an **MVP** quickly, helping you focus on business logic instead of spending all day setting up and writing boilerplate. So it might not be suitable for your team if you have many members and need a base project with standard **Clean Architecture**.

If you find it useful, don't hesitate to give me a star!

> GitHub Link: https://github.com/namanh11611/flutter_mvvm_riverpod
