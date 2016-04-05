---
title: 'Scheme实现的通用尾递归识别算法的demo实现,终于想到来些,拖延症...'
date: 2014-05-04 00:00:00
tags: Scheme
---
几个月前,我有写过一篇博客, [/2013/12/21/浅谈尾递归的定义和判定方法/](/2013/12/21/浅谈尾递归的定义和判定方法/) ,描述了一种用来识别简单和复杂尾递归的通用算法,后来还写了一篇博客描述了尾递归优化算法使用代码变换方式的一种实现算法(通过另外的基于continuation的计算模型也可以直接实现尾递归优化,因为那样没有了栈机制), 之后我一直想写代码去具体实现,终于,今天晚上QQ群灌水到没人在线时我打算来写一下了.

今天晚上只写了尾递归的识别算法的实现,而且没有做什么测试,写得匆忙,加上最近用Clojure,已经忘了scheme API怎么用了,所以,代码质量和鲁棒性不敢恭维,有问题看上面博客吧.


Github项目: [https://github.com/zoowii/tail-rec-optimization](https://github.com/zoowii/tail-rec-optimization)


先直接上代码([https://gist.github.com/zoowii/6932942e8661f8544ee3](https://gist.github.com/zoowii/6932942e8661f8544ee3))


	# lang racket
    ;; 这里是示例代码，这段代码在Scheme中是可以执行的，因为Scheme标准规定了需要尾递归优化
    
    
    ;; 而对应的JavaScript代码是无法运行的，因为没有做尾递归优化，很快就超过最大调用深度了
    
    
    (define (foo n)
    
    
     (if (> n 20140000)
    
    
        (begin
    
    
          (display "foo")
    
    
           n)
    
    
        (bar (+ n 1))))
    
    
    (define (bar n)
    
    
      (if (> n 20130000)
    
    
         (begin
    
    
            (display "bar")
    
    
             n)
    
    
         (foo (+ n 2))))
    
    
    (display (foo 1234))
    
    
     
    
    
    ;; 下面开始实际程序
    
    
    (define call/cc call-with-current-continuation)
    
    
     
    
    
    ;; 这里是上面的示例代码，program就是实际使用中的程序代码（宏展开后）
    
    
    (define program '((define (foo n)
    
    
     (if (> n 20140000)
    
    
        (begin
    
    
          (display "foo")
    
    
           n)
    
    
        (bar (+ n 1))))
    
    
    (define (bar n)
    
    
      (if (> n 20130000)
    
    
         (begin
    
    
            (display "bar")
    
    
             n)
    
    
         (foo (+ n 2))))
    
    
    (foo 1234) ; 这里没有加上display，是为了方便程序找到这个需要尾递归优化的函数foo
    
    
      ))
    
    
     
    
    
    (define (id form)
    
    
      ;; 辅助函数，一个返回参数自身的函数
    
    
      form)
    
    
     
    
    
    (define (procedure-definition? form)
    
    
      ;; 判断一个form是不是一个函数定义
    
    
      (if (and (list? form)
    
    
               (> (length form) 2)
    
    
               (eq? (car form) 'define)
    
    
               (list? (cadr form))
    
    
               (> (length (cadr form)) 1)
    
    
               )
    
    
          #t
    
    
          #f))
    
    
     
    
    
    (define (find-procedure-definitions program)
    
    
      ;; 找到一段程序中所有的函数定义的名称
    
    
      (filter id
    
    
              (map (lambda (form)
    
    
                     (if (procedure-definition? form)
    
    
                         (caadr form)
    
    
                         #f))
    
    
                   program)))
    
    
     
    
    
    (define (name-of-proc-definition proc-definition)
    
    
      ;; 在一个函数定义的代码中获取函数名称
    
    
      (caadr proc-definition))
    
    
     
    
    
    (define (find-func-calls program)
    
    
      ;; 找到一段程序中除了函数定义之外的函数调用，比如(foo 1234)
    
    
      (filter id
    
    
              (map (lambda (form)
    
    
                     (if (procedure-definition? form)
    
    
                         #f
    
    
                         form)) program)))
    
    
     
    
    
    (define (func-called program)
    
    
      ;; 找到一段程序中所有顶层被调用的函数,除了函数定义这类special form
    
    
      (map (lambda (form) (car form))
    
    
           (find-func-calls program)))
    
    
     
    
    
    (define (get-proc-definition program func)
    
    
      ;; 在一段程序中找到某个函数的定义代码
    
    
      (let ([procs (filter (lambda (f)
    
    
                            (equal? (name-of-proc-definition f)
    
    
                               func))
    
    
                          (filter procedure-definition?
    
    
                                  program))])
    
    
        (car procs)))
    
    
     
    
    
    (define (merge cols)
    
    
      ;; 合并一组列表
    
    
      (if (empty? cols)
    
    
          '()
    
    
          (let ([l1 (car cols)]
    
    
                [ll (cdr cols)])
    
    
            (if (empty? l1)
    
    
                (merge ll)
    
    
                (cons (car l1)
    
    
                      (merge (cons (cdr l1) ll)))))))
    
    
     
    
    
    (define (get-last-exprs forms)
    
    
      ;; 获取一个form集合中所有可能最后执行的form
    
    
      (merge
    
    
       (map (lambda (form)
    
    
             (cond  ;; 目前只考虑if和begin两种结构化,因为只是demo,如果是具体的编译器/解释器,自行获取最后可能执行的form
    
    
               [(not (list? form))
    
    
                (list form)]
    
    
               [(equal? 'if (car form))
    
    
                (if (> (length form) 3)
    
    
                    (get-last-exprs (list (caddr form)
    
    
                          (cadddr form)))
    
    
                    (get-last-exprs (list (caddr form))))]
    
    
               [(equal? 'begin (car form))
    
    
                (get-last-exprs (list (last form)))]
    
    
               [#t (list form)]))
    
    
           forms)))
    
    
     
    
    
    (define (get-body-of-proc-definition proc-definition)
    
    
      ;; 获取一个函数定义代码段中的body部分
    
    
      (cddr proc-definition))
    
    
     
    
    
    (define (base? form)
    
    
      ;; 判断一个form是否是基本类型,比如数值,字符串,布尔值,符号symbol
    
    
      ;; 也就是是否不是list
    
    
      (not (list? form)))
    
    
     
    
    
    ;;; 要记住在一个函数在一个函数集中尾递归依赖的函数
    
    
    (define (find-tail-rec-required-funcs func program)
    
    
      ;; 找到一个函数的尾递归依赖的函数集(就是尾部调用的函数,没有地柜调用)
    
    
      ;; 没有考虑更复杂的词法作用域,匿名函数等,这些由具体编译器/解释器的实现来判断
    
    
      (let* ([func-body (get-body-of-proc-definition
    
    
                        (get-proc-definition program func))]
    
    
            [last-exprs (get-last-exprs func-body)]
    
    
            [last-exprs-requirements (map
    
    
                                      (lambda (form)
    
    
                                        (if (base? form)
    
    
                                            #t
    
    
                                            (let ([item1 (car form)])
    
    
                                              (if (list? item1)
    
    
                                                  #f
    
    
                                                  item1))))
    
    
                                      last-exprs)])
    
    
        last-exprs-requirements))
    
    
     
    
    
    (define (into-set item col)
    
    
      ;; 在一个集合中添加一项,如果这个值已经存在,则不添加(也就是当做set)处理
    
    
      (if (member item col)
    
    
          col
    
    
          (cons item col)))
    
    
     
    
    
    (define (sub-set col1 col2)
    
    
      ;; 判断col1是否是col2的子集
    
    
      (let* ([diff (filter (lambda (x)
    
    
                             (not (member x col2)))
    
    
                           col1)])
    
    
        (empty? diff)))
    
    
     
    
    
    (define (find-requirements-col func program col C-col)
    
    
      ;; 在一个集合中找到依赖函数集
    
    
      ;; 过程就是找到尾部依赖的内容,进行判断,如果是一个函数调用,判断这个函数是否已经加入到col中,以及其他判断和操作
    
    
      ;; 如果不是尾递归的,返回#f, 如果是尾递归的, 返回#t, 如果依赖一个函数集,返回这个函数集
    
    
      ;; C-col记录依赖集的作用范围, col记录依赖的函数
    
    
      (let* ([last-exprs (find-tail-rec-required-funcs func program)])
    
    
        (if (member #f last-exprs)
    
    
            (list #f C-col)
    
    
            (let* ([exprs (filter (lambda (x) (and (not (boolean? x))
    
    
                                                  (not (member x col))
    
    
                                                  (not (equal? x func))))
    
    
                                  last-exprs)]
    
    
                   [new-col (merge (list col))]
    
    
                   [C-col (merge (list col exprs))]
    
    
                   [exprs-require (map (lambda (x)
    
    
                                         (find-requirements-col x program new-col C-col))
    
    
                                       exprs)]
    
    
                   [exprs-require (filter (lambda (x)
    
    
                                            (not (boolean? x)))
    
    
                                          (map car exprs-require))])
    
    
              (if (empty? exprs)
    
    
                  (list #t C-col)
    
    
                  (if (member #f exprs-require)
    
    
                  (list #f C-col)
    
    
                  (let* ([exprs-require (filter (lambda (rs)
    
    
                                               (not (sub-set rs new-col)))
    
    
                                             exprs-require)]
    
    
                         [new-col (merge (list (merge exprs-require) new-col))])
    
    
                    (if (and (= 1 (length new-col))
    
    
                             (equal? func (car new-col)))
    
    
                        (list #t C-col)
    
    
                        (list new-col C-col)))))))))
    
    
     
    
    
    (define (find=tail-rec-of-func program func)
    
    
      ;; 在一段程序中判断函数func是否是尾递归的,
    
    
      ;; 如果是,返回使尾递归成立的最小函数集合(范围),
    
    
      ;; 否则,返回nil
    
    
      (find-requirements-col func program (list func) (list func)))
    
    
     
    
    
    (define (println . args)
    
    
      (begin
    
    
        (map (lambda (x) (display x)) args)
    
    
        null))
    
    
     
    
    
    (define (find-tail-rec program)
    
    
      ;; 找到一段程序中的所有尾递归
    
    
      ;; 目前为了简单起见，而且demo代码中顶层只有一个函数调用(foo 1234)。所以只考虑最后一个函数调用，作为要判断尾递归优化的目标
    
    
      (let* ([funcs (func-called program)]
    
    
            [rec-states (map (lambda (func)
    
    
                               (find=tail-rec-of-func program func))
    
    
                             funcs)])
    
    
        (display "函数调用列表(实际被调用了的函数,没被调用的函数不通过转换代码实现尾递归优化):\n=======\n")
    
    
        (map (lambda (func)
    
    
               (begin
    
    
                 (display func)
    
    
                 (newline)))
    
    
             funcs)
    
    
        (display "=======\n")
    
    
        (map (lambda (func answer)
    
    
               (begin
    
    
                 (println "--- 函数 " func " ---\n")
    
    
                 (if (equal? #t (car answer))
    
    
                     (begin
    
    
                       (println "可以尾递归优化,在函数集 " (cadr answer) "中\n"))
    
    
                     (if (equal? #f (car answer))
    
    
                         (println "不可以尾递归优化\n")
    
    
                         (begin
    
    
                           (println "不可以尾递归优化,依赖函数集 " (cadr answer "\n")))))))
    
    
             funcs rec-states)
    
    
        null))
    
    
     
    
    
    ;; (display (func-called program))
    
    
    ;; (newline)
    
    
    ;; (define foo-def (get-proc-definition program 'foo))
    
    
    ;; (define foo-body (get-body-of-proc-definition foo-def))
    
    
    ;; (define foo-last-forms (get-last-exprs foo-body))
    
    
    ;; (display foo-last-forms)
    
    
    ;; (newline)
    
    
     
    
    
    ;; (display (find-tail-rec-required-funcs 'foo program))
    
    
    ;; (newline)
    
    
     
    
    
    ;; (display (find-requirements-col 'foo program '(foo) '(foo)))  ;; => '(#t (foo bar))
    
    
     
    
    
    (println (find-tail-rec program))


代码在Racket下得执行结果如下:

![](http://zoowiipublicstore.qiniudn.com/tail-recur-recognization%E6%89%A7%E8%A1%8C%E7%BB%93%E6%9E%9C%E6%88%AA%E5%9B%BE.png)



算法的具体描述看上面提到的博客.
