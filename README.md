# Lab3
## Протокол Ди́ффи — Хе́ллмана
### Протокол Ди́ффи — Хе́ллмана (англ. Diffie–Hellman key exchange protocol, DH) — криптографический протокол, позволяющий двум и более сторонам получить общий секретный ключ, используя незащищенный от прослушивания канал связи. Полученный ключ используется для шифрования дальнейшего обмена с помощью алгоритмов симметричного шифрования. Данный алгоритм позволил дать ответ на главный вопрос: «Как при обмене зашифрованными посланиями уйти от необходимости передачи секретного кода расшифровки, который, как правило, не меньше самого послания?» Открытое распространение ключей Диффи — Хеллмана позволяет паре пользователей системы выработать общий секретный ключ, не обмениваясь секретными данными.
#### Предположим, существует два абонента: Алиса и Боб. Алиса и Боб могут обмениваться сообщениями по каналу, который прослушивается. Любое их сообщение перехватывается Евой. Секрет состоит в выборе общего секретного ключа, и чтобы Ева не получила его копию. 
#### Обоим абонентам известны некоторые два числа g и p, которые не являются секретными и могут быть известны также Еве (g = первообразный корень по модулю р. g = 3, p = открытое простое число. p = 17).
        a_public=3

        b_public=17

        message="This is a very secret message!!!"

        Alice = DH_Endpoint(a_public, b_public, a_private)

        Bob = DH_Endpoint(a_public, b_public, b_private)

#### Алиса выбирает случайное приватное число, a = секретный ключ Алисы. a = 54.

        a_private=54
        
#### Затем Алиса вычисляет остаток от деления:

![2023-01-19_20-35-31](https://user-images.githubusercontent.com/122459067/213518696-ddd1685a-cc80-4916-b36e-8e27d8efb366.png)

        def generate_partial_key(self):

        partial_key = self.public_key1**self.private_key
        
        partial_key = partial_key%self.public_key2
        
        return partial_key
        
#### и пересылает этот результат публично Бобу:

        a_partial=Alice.generate_partial_key()

        print(a_partial) #15

#### Боб выбирает свое приватное число, b = секретный ключ Боба. b = 24.

        b_private=24

#### И вычисляет остаток от деления:
  
![2023-01-19_20-34-26](https://user-images.githubusercontent.com/122459067/213518527-52ec4268-012a-4091-9c2c-e12027a8e23f.png)

#### и передаёт Алисе:

        b_partial=Bob.generate_partial_key()
        
        print(b_partial) 
        
#### Предполагается, что Ева может получить оба этих значения, но не модифицировать их (то есть, у нее нет возможности вмешаться в процесс передачи).
#### Теперь, когда Алиса и Боб успешно обменялись частичными ключами, снова применим оператор «возведения в степень» и «деления по модулю» (power и modulo) к частичному ключу другого человека, используя собственные закрытые ключи следующим образом:

        def generate_full_key(self, partial_key_r):
        
        full_key = partial_key_r**self.private_key
        
        full_key = full_key%self.public_key2
        
        self.full_key = full_key
        
        return full_key
        
#### Далее Алиса берет публичный результат Боба и возводит его в степень своего приватного числа:

![2023-01-19_20-36-23](https://user-images.githubusercontent.com/122459067/213518908-360eb9d9-3d5c-4d7f-8dec-45586d028d0d.png)

#### И получает общий секретный ключ: 1.

        a_full=Alice.generate_full_key(b_partial)
        
        print(a_full) #1
        
#### Боб берет публичный результат Алисы и возводит его в степень своего приватного числа:

![2023-01-19_20-37-02](https://user-images.githubusercontent.com/122459067/213519013-45f0545d-227b-40b9-a7eb-0d7671321bd2.png)

#### И получает тот же самый общий секретный ключ: 1.

        b_full=Bob.generate_full_key(a_partial)
        
        print(b_full) #1
        
#### Оба они произвели одинаковые вычисления, только степени были в разном порядке.

#### Теперь, когда есть общий ключ шифрования, пришло время начать обмен сообщениями. Простой алгоритм шифрования, чтобы зашифровать сообщение Боба:

        def encrypt_message(self, message):
        
        encrypted_message = ""
        
        key = self.full_key
        
        for c in message:
        
        encrypted_message += chr(ord(c)+key)
        
        return encrypted_message
        
#### Каждый символ кодируется в целое число, добавляется значение ключа, а затем целое число преобразуется обратно в символ. Этот процесс повторяется для каждого символа в сообщении.

        b_encrypted=Bob.encrypt_message(message)
        
        print(b_encrypted) #'\x9f³´¾k´¾k¬kÁ°½Äk¾°®½°¿k¸°¾¾¬²°lll'
        
#### Теперь пришло время расшифровать сообщение:

        def decrypt_message(self, encrypted_message):
        
        decrypted_message = ""
        
        key = self.full_key
        
        for c in encrypted_message:
        
        decrypted_message += chr(ord(c)-key)
            
        return decrypted_message
        
#### Эта функция делает обратное. В то время как шаг шифрования добавляет значение ключа, шаг расшифровки вычитает то же значение ключа.

        message = Alice.decrypt_message(b_encrypted)
        
        print(message) #'This is a very secret message!!!'
        
#### Общая схема работы алгоритма:
 
 ![2023-01-19_19-37-31](https://user-images.githubusercontent.com/122459067/213519116-8d5eb102-d481-4221-8228-dae0eb1e56f2.png)
 
#### Базовый класс ниже конечной точки, использующей шифрование ДХ:

     class DH_Endpoint(object):        
     def __init__(self, public_key1, public_key2, private_key):            
        self.public_key1 = public_key1                
        self.public_key2 = public_key2                
        self.private_key = private_key                
        self.full_key = None                
    def generate_partial_key(self):
        partial_key = self.public_key1**self.private_key
        partial_key = partial_key%self.public_key2
        return partial_key
    def generate_full_key(self, partial_key_r):
        full_key = partial_key_r**self.private_key
        full_key = full_key%self.public_key2
        self.full_key = full_key
        return full_key
    def encrypt_message(self, message):
        encrypted_message = ""
        key = self.full_key
        for c in message:
            encrypted_message += chr(ord(c)+key)
        return encrypted_message
    def decrypt_message(self, encrypted_message):
        decrypted_message = ""
        key = self.full_key
        for c in encrypted_message:
            decrypted_message += chr(ord(c)-key)
        return decrypted_message


