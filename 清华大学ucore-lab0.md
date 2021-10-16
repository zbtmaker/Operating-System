# 清华大学ucore-lab0

## lab0_ex1

```c
int count = 1;
int value = 1;
int buf[0];
int main() {
	asm(
		"cld \n\t"
			"rep \n\t"
			"stosl"
		:
		: "c" (count), "a" (value), "D" (buf[0])
		:
		);
	return 0;
}
```

执行下面的命令后输出lab0_ex.s文件



lab0_ex.s文件

```c
	.section	__TEXT,__text,regular,pure_instructions
	.build_version macos, 10, 15	sdk_version 10, 15, 6
	.globl	_main                   ## -- Begin function main
	.p2align	4, 0x90
_main:                                  ## @main
	.cfi_startproc
## %bb.0:
	pushl	%ebp
	.cfi_def_cfa_offset 8
	.cfi_offset %ebp, -8
	movl	%esp, %ebp
	.cfi_def_cfa_register %ebp
	pushl	%edi
	pushl	%eax
	.cfi_offset %edi, -12
	calll	L0$pb
L0$pb:
	popl	%eax
	movl	L_buf$non_lazy_ptr-L0$pb(%eax), %ecx
	movl	$0, -8(%ebp)
	movl	_count-L0$pb(%eax), %edx
	movl	_value-L0$pb(%eax), %eax
	movl	(%ecx), %edi
	movl	%edx, %ecx
	## InlineAsm Start
	cld
	rep
	stosl	%eax, %es:(%edi)
	## InlineAsm End
	xorl	%eax, %eax
	addl	$4, %esp
	popl	%edi
	popl	%ebp
	retl
	.cfi_endproc
                                        ## -- End function
	.section	__DATA,__data
	.globl	_count                  ## @count
	.p2align	2
_count:
	.long	1                       ## 0x1

	.globl	_value                  ## @value
	.p2align	2
_value:
	.long	1                       ## 0x1

	.comm	_buf,1,2                ## @buf
	.section	__IMPORT,__pointers,non_lazy_symbol_pointers
L_buf$non_lazy_ptr:
	.indirect_symbol	_buf
	.long	0

.subsections_via_symbols
```



[X86汇编语言指令文档](https://www.aldeid.com/wiki/X86-assembly/Instructions/mov)

