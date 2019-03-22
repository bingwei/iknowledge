C++的早期是C with Class，一个对象实际上就是一个C的struct。例如，一个stack类

```C++
class stack {
    char s[10];
    char* min;
    char* top;
    char* max;
    void new();
public:
    void push();
    char pop();
};

void stack.push(char c)
{
    if (top>max) 
        error("stack overflow");
    *top++ = c;
}

void g(class stack* p)
{
    p->push('c');
}
```

转换后的结果是

```C
struct stack { /* generated C code */
    char s[10];
    char* min;
    char* top;
    char* max;
};

/* generated C code */
void stack push(struct stack* this, char c)
{
    if ((this->top)>(this->max)) 
        error("stack overflow");
    *(this->top)++ = c;
}

/* generated C code */
void g(struct stack* p)
{
    stack push(p,'c');
}
```

参考：The Design and Evolution of C++, p.38
