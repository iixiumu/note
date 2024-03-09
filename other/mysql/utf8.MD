1.MySQL的“utf8mb4”是真正的“UTF-8”。
2.MySQL的“utf8”是一种“专属的编码”，它能够编码的Unicode字符并不多。
我要在这里澄清一下：所有在使用“utf8”的MySQL和MariaDB用户都应该改用“utf8mb4”，永远都不要再使用“utf8”。