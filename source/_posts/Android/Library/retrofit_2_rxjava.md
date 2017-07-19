---
layout: post
title: Retrofit开发-2-结合RxJava
categories:
  - Android
  - Library
tags:
  - Android
  - Retrofit
keywords:
  - Android
  - Retrofit
  - OkHttp
  - RxJava2.x
comments: true
abbrlink: 36f21fa4
date: 2017-07-05 17:09:11
password:
---

请先阅读 [Retrofit 开发-1(OkHttp+Retrofit基本使用)](../e57450c6)

本文建立在对 `RxJava2.x` 和 `Retrofit` 基本熟悉的基础上，主要是在 `Android` 平台下对 `RxJava` + `Retrifit` 发送请求进行封装方法的探索。

<!--more-->

## 准备

首先你要确保在创建 `Retrofit` 时使用了 `RcJava` 的数据转换适配器。

依赖

```gradle
adapter_rxjava2 : 'com.jakewharton.retrofit:retrofit2-rxjava2-adapter:1.0.0',
```

配置

```
final Retrofit.Builder retrofitBuilder = new Retrofit.Builder();
retrofitBuilder.addCallAdapterFactory(RxJava2CallAdapterFactory.createWithScheduler(Schedulers.io()));
```

将数据中返回的全部类型改成 `Observable` 类型

```java
@GET(GET_USER_INFO)
Observable<UserInfoDTOResponse> getUser();

@GET(GET_BABY_RELATION_LIST)
Observable<UserBabyRelationResp> getBabyRelationList(@Query("babyId") Long babyId, @Query("limit") int limit, @Query("offset") Long offset);
```


## 请求数据

这边熟悉了 `RxJava` 的相关使用和操作符之后用起来还算得心应手，就不做过多解释，贴两段代码感受一下。

封装的 `Observer`

```java
public class ObserverWrap<T> implements Observer<T> {

    Context mContext;

    public ObserverWrap(Context context) {
        mContext = context;
    }

    public ObserverWrap() {
    }

    @Override
    public void onSubscribe(@NonNull Disposable d) {
        L.eWithTread("Rx", "onSubscribe");
    }

    @Override
    public void onNext(@NonNull T t) {
        L.eWithTread("Rx", "onNext -> " + t.toString());
    }

    @Override
    public void onError(@NonNull Throwable e) {
        L.eWithTread("Rx", "onError");
        e.printStackTrace();
        ToastUtil.show("请求失败");
        if (e instanceof UnknownHostException) {
            ToastUtil.show("网络不好");
        }
        onFinish();
    }

    @Override
    public void onComplete() {
        L.eWithTread("Rx", "onComplete");
        onFinish();
    }

    public void onFinish() {
        if (mContext != null) {
            ((BaoBaoBaseActivity) mContext).dismissLoadingDialog();
        }
    }
}
```
获取用户信息

```java
// 调用
RequestTest.getUserInfo()
        .subscribe(new ObserverWrap<UserInfoDTOResponse.UserInfoDTO>());
        
// 方法
public static Observable<UserInfoDTOResponse.UserInfoDTO> getUserInfo() {
    return ApiRequest.getService().getUser()
            .map(new Function<UserInfoDTOResponse, UserInfoDTOResponse.UserInfoDTO>() {
                @Override
                public UserInfoDTOResponse.UserInfoDTO apply(@io.reactivex.annotations.NonNull UserInfoDTOResponse userInfoDTOResponse) throws Exception {
                    return userInfoDTOResponse.getData();
                }
            })
            .observeOn(AndroidSchedulers.mainThread());
}
```

来一个复杂点的，获取一个列表

```java
// 调用
RequestTest.getUserBabyRelationList(mActivity,baby)
        .subscribe(new ObserverWrap<UserBabyRelation>(mActivity));
        
// 方法
public static Observable<UserBabyRelation> getUserBabyRelationList(final Context context, Baby baby) {
        return ApiRequest.getService()
                .getBabyRelationList(baby.getBabyId(), 100, null)
                .subscribeOn(Schedulers.newThread())
                .doOnSubscribe(new Consumer<Disposable>() {
                    @Override
                    public void accept(@io.reactivex.annotations.NonNull Disposable disposable) throws Exception {
                        if(context instanceof BaoBaoBaseActivity){
                            ((BaoBaoBaseActivity) context).showLoadingDialog();
                        }
                    }
                })
                .subscribeOn(AndroidSchedulers.mainThread())
                .filter(new Predicate<UserBabyRelationResp>() {
                    @Override
                    public boolean test(@io.reactivex.annotations.NonNull UserBabyRelationResp userBabyRelationResp) throws Exception {
                        if (userBabyRelationResp.getStatus() == 200 && isRespNotNull(userBabyRelationResp)) {
                            return true;
                        } else if (userBabyRelationResp.getStatus() != 200) {
                            if (userBabyRelationResp.getMessage() != null) {
                                ToastUtil.show(userBabyRelationResp.getMessage());
                            }
                            return false;
                        }
                        return false;
                    }
                })
                .map(new Function<UserBabyRelationResp, List<UserBabyRelation>>() {
                    @Override
                    public List<UserBabyRelation> apply(@io.reactivex.annotations.NonNull UserBabyRelationResp userBabyRelationResp) throws Exception {
                        return userBabyRelationResp.getData().getList();
                    }
                })
                .flatMap(new Function<List<UserBabyRelation>, ObservableSource<UserBabyRelation>>() {
                    @Override
                    public ObservableSource<UserBabyRelation> apply(@io.reactivex.annotations.NonNull List<UserBabyRelation> userBabyRelations) throws Exception {
                        return Observable.fromIterable(userBabyRelations);
                    }
                })
                .observeOn(AndroidSchedulers.mainThread());
}
```


## 封装

在 `Android` 平台下我们进行网络请求时，往往有些特殊的要求，总结如下，不断完善实现下列需求的方法，寻找一种更合适更简单的 `Retrofit` + `RxJava` 的网络请求架构。

> 1. 请求开始时，显示 `Dialog` 提示用户等待，请求结束时消掉 `Dialog`
> 2. 当 `Activity` 销毁时取消当前界面的请求。
> 3. 当 `Dialog` 被认为 `cancel` 时，取消当前页面的请求。

 
