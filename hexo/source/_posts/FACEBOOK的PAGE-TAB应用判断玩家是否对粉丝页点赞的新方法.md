---
title: FACEBOOK的PAGE TAB应用判断玩家是否对粉丝页点赞的新方法
date: 2015-03-30 00:00:0
tags:
---

Facebook的Page Tab应用在被访问时会收到一个signed_request参数，这个参数的字段说明可以参考https://developers.facebook.com/docs/reference/login/signed-request

其中page下有个liked字段，以前可以据此判断玩家是否对当前粉丝页点了赞，如今这个字段已经移除了。

Facebook总是搞这种事，尽管它有自己的想法，但是对于开发者来说，实在是灾难。

Google的服务用着用着就关了，Facebook的接口用着用着就变了。

那现在怎么判断玩家是否点赞了呢？

Facebook在这里给了一个方法：https://developers.facebook.com/docs/graph-api/common-scenarios

    Does someone like a Facebook Page?
    
    Although you can’t get a list of all the fans of a Facebook Page, you can find out whether a specific person has liked a Page.
    
    Use These APIs
    /{user-id}/likes/{page-id} – this API Read modifier will let you check via API.

这是Facebook的Graph API，我不知道这个能否长久用下去，但起码现在是可用的。

所以你需要发起

    https://graph.facebook.com/{user-id}/likes/{page-id}?access_token={access_token}
    
的请求。

这里面user-id、page-id、access_token都能从signed_request里拿到，分别是user_id、page.id和oauth_token字段。

当玩家点了赞时，返回的data参数里，有粉丝页的信息，若玩家未点赞，则data参数里，没有数据。

据此判断即可。

不过想使用这个API的前提是，应用获得了玩家的user_likes权限。

为了获得这个权限，我这边的处理方法是，新做一个中间页，页面里使用Facebook的JS SDK重新登录授权一下，登录后跳转到应用地址。

    FB.login(function(response) {
    //跳转到真正的应用地址
    }, {scope: 'user_likes'});

然后在开发者后台Page Tab的设置页面里，把URL改到那个中间页上就行了。

这样，原本的应用接收到signed_request时，已经取得了user_likes权限，就可以使用那个Graph API了。
