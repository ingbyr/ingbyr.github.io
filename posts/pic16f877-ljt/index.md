# 基于PIC16F877单片机的智能垃圾桶


> 微芯2016北邮奖学金获奖作品

# 视频演示

<iframe height=498 width=510 src="https://player.youku.com/embed/XMTM5MjQ2NzQ4MA==" frameborder=0 'allowfullscreen'></iframe>

# 汇编部分
```bash
		status	equ			03h
		portc		equ		07h
		trisc		equ		87h
		portd		equ		08h	;hw,smoke
		trisd		equ		88h
		porta		equ		05h 	
		trisa		equ		85h

		porte		equ		09h	;re0蜂鸣器 低有效
		trise		equ		89h			
			

		;keyborad
		portb		equ		06h
		trisb		equ		86h

		;=================定时器tmr0===================
		tmr0		equ		01h
		option_reg	equ		81h
		intcon		equ 		0bh
		tmr0b_fast	equ 		128
		tmr0b_slow	equ		0

		;==================常量定义===================
		count 		equ	21h	;电机转数
		ljt_status	equ	22h 	;桶盖状态，0为关闭，1为打开状态
		jp_status	equ	23h	;键盘状态
		yw_status  	equ	24h 	;yw

		;===================体1设置=========================
			org		000h
			nop
			bsf		status,5
			bcf 		status,6	;转到体1
			movlw		00h		;电机四位信号输出	
			movwf		trisc
			movlw		0ffh
			movwf		trisd		;红外信号一低两位输入,0为遮挡
			movlw		0ffh		;设置ra为输入
			movwf		trisa	
			movlw		0ffh		;rb输入	
			movwf		trisb
			
			movlw 		00h		;蜂鸣器输出
			movwf		trise

			movlw		03h 		;分频数
			movwf		option_reg
			bcf		option_reg,3	;分频器分配给tmr0
			bcf		option_reg,4	;上升沿tmr0递增
			bcf		option_reg,5	;内部时钟提供时钟源

			bcf		option_reg,7	;启用b端口上拉电阻
			goto		init

		;=====================体0设置=====================
		init
			bcf		status,5	;转到体0
			movlw		00h		;输出初始化	
			movwf		portc	
			movwf		ljt_status	;桶盖初始状态为0

			movlw		0ffh		;蜂鸣器初始不工作
			movwf		porte

			movlw		01dh		;电机初始转数
			movwf		count
			nop
			nop
			goto 		main

		;======================主程序====================
		main
			call		scan		;调用键盘扫描
			call		yw_gy

			movlw		01dh		;电机初始转数
			movwf		count
			btfsc		portd,0
			goto 		work1
			btfss		ljt_status,0	;若桶盖为1,跳过正转
			call		djzz		;电机正转

		work1
			movlw		01dh		;电机初始转数
			movwf		count
			btfss		portd,0
			goto		main
			btfsc		ljt_status,0	;若桶盖为0,跳过反转
			call		djfz		;电机反转
			goto		main


		;==============键盘扫描===============
		scan
		;第一行扫描	
			movlw		B'11111110'
			movwf		portb
			nop
			nop
			bsf		status,5	;到体1，转换方向
			movlw		0f0h
			movwf		trisb
			bcf		status,5	;返回体0
			movf		portb,0
			movwf		jp_status	;键盘状态读入通用寄存器
			btfss		jp_status,4
			goto		set_flag1
			btfss		jp_status,4
			goto		scan
			btfss		jp_status,5
			goto		set_flag2
		;第二行扫描
			bsf		status,5	;到体1，转换方向
			movlw		0fh
			movwf		trisb
			bcf		status,5	;返回体0
			movlw		B'11111101'
			movwf		portb
			nop
			nop
			bsf		status,5	;转换方向
			movlw		0f0h
			movwf		trisb
			bcf		status,5	;返回体0
			movf		portb,0
			movwf		jp_status
			btfss		jp_status,4
			goto		set_flag3
			btfss		jp_status,5
			goto		set_flag4
			return

		;============标志测试=============
		set_flag1				;电机正转打开桶盖
			movlw		01dh		;电机初始转数
			movwf		count
			call		djzz
			bsf		ljt_status,0
			return



		set_flag2				;电机反转关闭桶盖进入工作状态
			movlw		01dh		;电机初始转数
			movwf		count
			call		djfz
			bcf		ljt_status,0
			return



		set_flag3	;无
				
			return

		set_flag4	;无

			return
			
		;=====================电机正转=====================
		djzz
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			movlw 	080h
			movwf 	portc
			call	delay_fast
			call	delay_fast
			call	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			movlw 	0c0h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			movlw 	040h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			movlw 	060h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			movlw 	020h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			movlw 	030h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			movlw	010h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			movlw 	090h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			decfsz 	count,1
			goto 	djzz
			bsf	ljt_status,0	;桶盖打开标志1
			return	

		;======================电机反转====================
		djfz
			call	 delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			movlw 	090h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			movlw 	010h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			call 	delay_fast
			movlw 	030h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			movlw 	020h
			movwf 	portc
			call 	delay_fast
			call	delay_fast
			movlw 	060h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			movlw 	040h
			movwf 	portc
			call 	delay_fast
			call 	delay_fast
			movlw 	0c0h
			movwf 	portc  
			call 	delay_fast
			call 	delay_fast
			movlw 	080h
			movwf	portc
			call	 delay_fast
			call 	delay_fast
				call 	hwgy_tg_open  ;检测满，是则蜂鸣
			
			decfsz	count,1
			goto	djfz	
			bcf	ljt_status,0	;桶盖关闭0
			call 	delay_fast
			call 	hwgy_tg_close	;检测清空
			return	

		;==============红外感应===================	
		;红外感应使桶盖打开 rd0 
		hwgy_open
			call	delay_slow
			call	delay_slow
			call	delay_slow	
			btfsc	portd,0
			goto 	hwgy_open
			return 

		;红外感应使桶盖关闭	rd0
		hwgy_close
			call	delay_slow
			call	delay_slow
			call	delay_slow	
			btfss	portd,0
			goto 	hwgy_close
			return 

		;红外感应范围内是否有目标
		hwgy_tg_open
			btfss	portd,1
			call 	fmq_open
			return
			
		hwgy_tg_close
			btfsc 	portd,1
			call 	fmq_close
			return
			

		;==============蜂鸣器====================
		;蜂鸣器工作
		fmq_open
			movlw	00h
			movwf	porte
			return
		;蜂鸣器停止工作
		fmq_close
			movlw	0ffh
			movwf   porte
			return
			
		;================短延时===================
		delay_fast
			bcf	intcon,2
			movlw	tmr0b_fast
			movwf	tmr0
		loop1
			btfss	intcon,2
			goto	loop1
			return	

		;===============长延时===================
		delay_slow	
			bcf	intcon,2
			movlw	tmr0b_slow
			movwf	tmr0
		loop2
			btfss	intcon,2
			goto	loop2
			return	


		;===========yw===============

		yw_gy
			btfss	portd,4
			call	fmq_open
			btfsc	portd,4
			call	fmq_close
			return


			end
```


---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/pic16f877-ljt/  

