---
layout: post
title: PASSWORD_HASH
categories: php
---


原舊系統使用生日為密碼並存為明碼，考量到安全性的不足，使用了PHP 使用內建函式

因舊系統需要比對資料庫密碼，以下為時做邏輯

1. 寫入密碼前先用  `password_hash`+ 自訂 `salt`  加密密碼 
2. 將密碼及自訂 `salt` 寫入資料表

```php
$salt = creatSalt(22); // 鹽值最少22個字
$password = password_hash('使用者密碼', PASSWORD_BCRYPT, array('salt' => $salt));

/**
 * 建立hash的鹽值
 * @param $num
 * @author maggie 2021/10/1 上午 11:31
 */
private function creatSalt($num)
{
    $range = array();
    $range = array_merge($range, range('a','z'));
    $range = array_merge($range, range('A','Z'));
    $range = array_merge($range, range('0','9'));
    shuffle($range);
    $rangeArr = array_rand($range, $num);
    $salt = implode('', array_map(function ($v) use ($range) {
        return $range[$v];
    }, $rangeArr));

    return $salt;

}
```

1.  登入時取出自訂 `salt` ，再做一次  `password_hash`
2. 與資料表密碼比對成功者可順利登入

```php
// 取得學生資料
$studentData = $this->getModel('Overseapaymentmgr/StudentPayment')->fetchRow(array('email = ?' => $postData['email'], 'os_program_sn = ?' => $programInfo[F_SN]));

// 再加密一次密碼
$postData['password'] =  password_hash($postData['password'], PASSWORD_BCRYPT, array('salt' => $studentData->salt));
```

參考資料

[https://www.php.net/manual/zh/function.password-hash](https://www.php.net/manual/zh/function.password-hash)

[https://hoohoo.top/blog/modern-php-password-hash-hash-encryption-using-random-salt-uses/](https://hoohoo.top/blog/modern-php-password-hash-hash-encryption-using-random-salt-uses/)