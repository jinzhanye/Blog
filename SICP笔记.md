# SICP笔记
## 第一章

### 应用序(applicative order)与正常序(normal order)
#### 应用序
- 过程：不管该值是否用到，都会先进行求值(eval)，后进行运算(apply)
- example
````

````
#### 正常序
- 过程：不断进行替代模型，直到需要用到该值时才进行求值(eval)。
- example:
````

````
### 黑盒抽象

### 高阶过程(higher order)
将过程当作形参，或该过程返回一个过程，这样的过程称过高阶过称


### 迭代(linear iterative)与递归(recrusive)

#### 形状
，树形递归

| | 时间复杂度 | 空间复杂度 | 优点 | 缺点
|--- | ---|--- |--- |---
|迭代  | O(1) | O(n) | 可以保存中间结果 |  写出迭代表达式是有难度的
|递归  | O(n) | O(n) | 直观 | 1.不会保存中间结果2.产生多余的计算 - 解决方法：利用一个表保存计算中间值。计算前先在表中查找结果，如果有结果则直接取结果，如果没有则进行计算，然后将结果保存到表中。

#### C、Java,Javascript等其他语言实现迭代的方式
for循环，尾递归优化

## 第二章
- 闭包性质
- double,求导等应用
- 加一种cons的实现方式
````
(define (cons x y)
  (define (dispatch m)
    (cond ((= m 0) x)
          ((= m 1) y)
	      (else (error "Argument not 0 or 1: CONS" m))))
  dispatch)

(define (car z) (z 0))
(define (cdr z) (z 1))
````
- **用愿望思维写程序**
思想：过程也被看作成数据
- 利用数据抽象进行分数操作
````
;*****分数数据抽象
(define (equla-rat? x y)
  (= (* (numer x)(denom y))
   (* (numer y)(denom x))))

;*****分数运算
;加
(define (add-rat x y)
  (make-rate (+ (* (numer x) (denom y))
			    (* (numer y) (denom x)))
			 (* (denom x) (denom y))))

;减
(define (sub-rat x y)
  (make-rate (- (* (numer x) (denom y))
			    (* (numer y) (denom x)))
			 (* (denom x) (denom y))))	
;乘
(define (mul-rat x y)
  (make-rate (* (numer x) (numer y))
			 (* (denom x) (denom y))))
;除
(define (div-rat x y)
  (make-rate (* (numer x) (denom y))
			 (* (denom x) (numer y))))		

;*****化简可以在 
;*****(1)进行操作运算后保存数据时进行，
;*****(2)可以在取出数据时进行，根据具体业务选择不同的化简方式。
;简化版
(define (make-rate n d)
  (let ((g (gcd n d)))
    (cons (/ n g) (/ d g))))

(define (numer x)
  (car x))

(define (denom x)
  (cdr x))

(define one-half (make-rate 1 2))
(define one-third (make-rate 1 3))

; (print-rat (add-rat one-half one-third))
; (print-rat (add-rat one-third one-third))

;**********2.12
;原始未简化版
(define (make-rate n d) (cons n d))
;在取分子、分母时化简
(define (numer x)
  (let ((g (gcd (car x) (cdr x))))
    (/ (car x) g)))

(define (denom x)
  (let ((g (gcd (car x) (cdr x))))
    (/ (cdr x) g)))

(print-rat (add-rat one-third one-third))
````


- filter实现
````
(define (filter predicate sequence)
  (cond ((null? sequence) nil)
        ((predicate (car sequence)) 
          (cons (car sequence)
	             (filter predicate (cdr sequence))))
        (else (filter predicate (cdr sequence)))))
````
- 递归实现map
````
(define (map proc items)
  (if (null? items)
      items
      (cons (proc (car items))
            (map proc (cdr items)))))
````