# Compare sentinel check vs ifs catch all

## Anylysis 

Well optimized versions for both examples have slightly different characteristis. If branch prediction guesses correctly the jumps in `normal.c` would perform slightly faster, but much slower if one, or the more, of the predictions fail. While `sentinal.c` would be slower in some cases, it would faster and more consistent when receiving unbiased input. 

## normal.c

```
int function(int value) {
  if (value == 0) {
    return 0;
  }

  if (value == 1) {
    return 1;
  }

  return -1;
};
```

### non-optmized

```
0000000000000000 <function>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	89 7d fc             	mov    %edi,-0x4(%rbp)
   7:	83 7d fc 00          	cmpl   $0x0,-0x4(%rbp)
   b:	75 07                	jne    14 <function+0x14>
   d:	b8 00 00 00 00       	mov    $0x0,%eax
  12:	eb 12                	jmp    26 <function+0x26>
  14:	83 7d fc 01          	cmpl   $0x1,-0x4(%rbp)
  18:	75 07                	jne    21 <function+0x21>
  1a:	b8 01 00 00 00       	mov    $0x1,%eax
  1f:	eb 05                	jmp    26 <function+0x26>
  21:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  26:	5d                   	pop    %rbp
  27:	c3                   	retq   
```

### O1

```
0000000000000000 <function>:
   0:	b8 00 00 00 00       	mov    $0x0,%eax
   5:	85 ff                	test   %edi,%edi
   7:	74 0d                	je     16 <function+0x16>
   9:	83 ff 01             	cmp    $0x1,%edi
   c:	0f 94 c0             	sete   %al
   f:	0f b6 c0             	movzbl %al,%eax
  12:	8d 44 00 ff          	lea    -0x1(%rax,%rax,1),%eax
  16:	f3 c3                	repz retq 
```

### O2 & O3

```
0000000000000000 <function>:
   0:	31 c0                	xor    %eax,%eax
   2:	85 ff                	test   %edi,%edi
   4:	74 0c                	je     12 <function+0x12>
   6:	31 c0                	xor    %eax,%eax
   8:	83 ff 01             	cmp    $0x1,%edi
   b:	0f 94 c0             	sete   %al
   e:	8d 44 00 ff          	lea    -0x1(%rax,%rax,1),%eax
  12:	f3 c3                	repz retq 

```

## sentinal.c

```
int function(int value) {
  if (value != 0 && value != 1) {
    return -1;
  }

  if (value == 0) {
    return 0;
  }

  if (value == 1) {
    return 1;
  }
};
```

### non-optmized

```
0000000000000000 <function>:
   0:	55                   	push   %rbp
   1:	48 89 e5             	mov    %rsp,%rbp
   4:	89 7d fc             	mov    %edi,-0x4(%rbp)
   7:	83 7d fc 00          	cmpl   $0x0,-0x4(%rbp)
   b:	74 0d                	je     1a <function+0x1a>
   d:	83 7d fc 01          	cmpl   $0x1,-0x4(%rbp)
  11:	74 07                	je     1a <function+0x1a>
  13:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  18:	eb 1a                	jmp    34 <function+0x34>
  1a:	83 7d fc 00          	cmpl   $0x0,-0x4(%rbp)
  1e:	75 07                	jne    27 <function+0x27>
  20:	b8 00 00 00 00       	mov    $0x0,%eax
  25:	eb 0d                	jmp    34 <function+0x34>
  27:	83 7d fc 01          	cmpl   $0x1,-0x4(%rbp)
  2b:	75 07                	jne    34 <function+0x34>
  2d:	b8 01 00 00 00       	mov    $0x1,%eax
  32:	eb 00                	jmp    34 <function+0x34>
  34:	5d                   	pop    %rbp
  35:	c3                   	retq   
```

### O1

```
0000000000000000 <function>:
   0:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
   5:	83 ff 01             	cmp    $0x1,%edi
   8:	77 16                	ja     20 <function+0x20>
   a:	85 ff                	test   %edi,%edi
   c:	74 07                	je     15 <function+0x15>
   e:	83 ff 01             	cmp    $0x1,%edi
  11:	74 08                	je     1b <function+0x1b>
  13:	f3 c3                	repz retq 
  15:	b8 00 00 00 00       	mov    $0x0,%eax
  1a:	c3                   	retq   
  1b:	b8 01 00 00 00       	mov    $0x1,%eax
  20:	f3 c3                	repz retq 
```

### O2 & O3

```
0000000000000000 <function>:
   0:	83 ff 01             	cmp    $0x1,%edi
   3:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
   8:	0f 46 c7             	cmovbe %edi,%eax
   b:	c3                   	retq   

```