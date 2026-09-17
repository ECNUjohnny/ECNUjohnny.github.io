+++

date = "2026-09-16"
author = "Johnny"
title = "cpp compile knowledge again"
tags = ["c++"]

+++

```
char *str;

int a, b, c;
int offset = 0, chars = 0;

while (ssccanf(str + offset, "%d/%d/%d%n", &a, &b, &c, &chars))
{
    offset += chars;
}

```

if we want to read data like "1/2/3 1/2/3 1/2/3" and extract the number included in it, we can use codes like above. 

we can use "%n" in sscanf function to record how many chars we have processed(including those chars that are not actually readed, like " ").

"sscanf" allows you to read specific format of datas from a string. It will only skip chars like " ", "\t", so we should put the first pointer at a chars like these or the chars we want to read