---
title: "UMDCTF2024"
date: 2024-05-02T00:26:40+09:00
draft: true
---

# rev

### cmsc430

```C
undefined8 main(void)

{
  int iVar1;
  undefined8 uVar2;
  
  in = _stdin;
  out = _stdout;
  error_handler = error_exit;
  uVar2 = entry();
  print_result(uVar2);
  iVar1 = val_typeof(uVar2);
  if (iVar1 != 4) {
    putchar(10);
  }
  return 0;
}
```

```entry()``` を見ると，大量の if else が見つかる．

```C
  *(undefined8 *)((long)&pcStack_10 - ((ulong)&stack0xfffffffffffffff8 & 8)) = 0x1017f1;
  pcStack_10 = (code *)read_byte();
  if (((ulong)pcStack_10 & 1) == 0) {
    lVar1 = 7;
    if (pcStack_10 == (code *)0xaa) {
      lVar1 = 3;
    }
    if (lVar1 == 7) {
      uVar2 = 7;
    }
    else {
      *(undefined8 *)((long)&pcStack_10 - ((ulong)&stack0xfffffffffffffff8 & 8)) = 0x101849;
      pcStack_10 = (code *)read_byte();
      if (((ulong)pcStack_10 & 1) != 0) goto raise_error_align;
      lVar1 = 7;
      if (pcStack_10 == (code *)0x9a) {
        lVar1 = 3;
      }
      if (lVar1 == 7) {
        uVar2 = 7;
      }
      else {
        *(undefined8 *)((long)&pcStack_10 - ((ulong)&stack0xfffffffffffffff8 & 8)) = 0x1018a1;
        pcStack_10 = (code *)read_byte();
        if (((ulong)pcStack_10 & 1) != 0) goto raise_error_align;
        lVar1 = 7;
        if (pcStack_10 == (code *)0x88) {
          lVar1 = 3;

~~ snip ~~
```

フラグっぽい hex が見つかるので，これと ```read_byte()``` の中身を見て復元する．

###
