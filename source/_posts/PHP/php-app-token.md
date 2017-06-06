---
title: 我是怎么做App token认证的
date: 2017-06-06 18:30:00
category: [Web]
tags: [PHP]
toc: true
---

使用Token来做身份认证在目前的移动客户端上非常流行，Token这个概念来源于OAuth认证，主要是在服务端实现。关于相关的原理，同学们自行百度。在这里，我简单介绍一下我是怎么具体实现的，重点描述token生成、token识别及token缓存。

## 生成Token
服务端接收客户端传递的username和password等请求，在数据库中检查，如果用户名密码匹配的话，表示登录成功，服务端生成并返回一个token访问令牌。

```php
    public function login()
    {
        $data = array_merge($this->request->post(), []);

        // login with password
        $user = Db::name($this->table)->field($this->field_except, true)->where('name', $data['name'])->find();
        if ($user) {
            if ($user['pwd'] === EncryptService::password($data['pwd'])) {
                $this->onLoginSuccess($user);
            } else {
                parent::logic_failure('密码不正确', 'user_wrong_pwd');
            }
        } else {
            parent::logic_failure('用户不存在', 'user_not_exists');
        }
    }

    private function onLoginSuccess($user)
    {
        unset($user['pwd']);
        $token = Token::generateToken($this->request, $user);
        $ret = [
            'token' => $token['token'],
            'user' => $user
        ];
        $this->logic_success(null, $ret);
    }

```
在上面的代码中，登录成功，服务端向客户端返回token和user信息。token是通过Token类的静态方法`generateToken`来生成的。

```php
    public static function generateToken($request, $user)
    {
        // 1，生成包含用户及设备信息的新token
        $data = [
            'uuid' => $request->param('uuid', ''),
            'uid' => $user['id'],
            'ip' => $request->ip(),
            'client' => $request->param('client', ''),
            'update_time' => ApiBase::_timestamp(),
        ];
        // json序列化
        $json = json_encode($data);
        // 加密token信息
        $key = md5(EncryptService::encrypt($json));
        $data['token'] = $key;
        // 2，从数据库查询用户的token，决定是update还是insert
        $token = Token::get($user['id']);
        // 如果数据库中已存在token，更新
        if ($token) {
            // 删除旧的缓存
            cache($token['token'], null);
            // 执行更新
            $token->isUpdate(true)->save($data);
        }
        // 不存在，插入
        else {
            $token = new Token();
            $data['create_time'] = ApiBase::_timestamp();
            $token->data($data)->save();
        }
        // 3，将token写入缓存
        Token::cacheToken($token->getData());
        return $token->getData();
    }
```

这里主要有3个步骤

1. 根据用户及设备等信息生成一个唯一的token，uid用于识别用户，uuid用于识别设备，time用于保证唯一。将这些信息序列化后加密并生成md5
2. 根据用户id查询用户登录表，不管用户原来是否已经存在token，新的token将替换旧的token
3. 将token写入缓存，因为token是每个请求都会解析，如果不使用缓存的话，会导致数据库访问瓶颈。

客户端应该保存token，然后在访问一些需要身份认证的API，比如修改昵称，就需要带上这个token了，而不需要显示带上用户信息，比如user_id。

## 解析token
服务端接收客户端传递的token，需要从中解析出相关的用户及设备信息。
```php
    public static function getToken($key)
    {
        if (!isset($key)) {
            return null;
        }
        $token = cache($key);
        if ($token) {
            return $token;
        }
        else {
            return Token::findByToken($key);
        }
    }
```
过程很简单，先从缓存中取，如果不存在再从数据库中查询。所有的请求，服务端都会执行解析token。

## 验证token
服务端简单地把API分为三类

1. 公共API，客户端可以任意访问，不需要验证身份，此类API多以GET请求为多。比如查询产品列表
2. 受保护的API，也即需要强制认证的API，这类API需要验证客户端身份才能访问，比如修改头像，客户端未传token或token无效，服务端返回一个错误码。
3. 可选认证的API，介于1和2之间，不要求强制验证客户端身份，比如分享奖励，如果客户端传递了token并认证通过了，则给相应的奖励，否则不做任何的奖励。

基于上面的分析，验证token附带一个是否强制验证的参数，用于中断。验证的规则也相对简单，只要用户请求的设备ID与token中的设备ID相同就算验证通过了。

```php
    public function checkToken($exit = true)
    {
        if ($this->token) {
            if ($this->token['uuid'] === $this->request->param('uuid', '')) {
                return true;
            } else {
                if ($exit) {
                    $this->logic_failure(null, 'user_offline');
                } else {
                    $this->token = null;
                }
                return false;
            }
        } else {
            if ($exit) {
                $this->logic_failure(null, 'token_invalid');
            }
            return false;
        }
    }
```

## 缓存

缓存相对简单，缓存的key为加密token之后的md5值，也即登录成功后，服务端向客户端发送的token值。缓存未设置有效期，默认永不过期。

```php 
    public static function cacheToken($token)
    {
        if ($token) {
            if (is_object($token)) {
                $token = $token->getData();
            }
            cache($token['token'], ($token));
        }
    }
?>
```

## 进阶

为了让您的应该更加安全，还可以在以下方面强化

1. token有效期
2. token缓存加密
3. token高级校验

不过，我这里已经对token做了对称加密，相信他人伪造token的可能性也不大。