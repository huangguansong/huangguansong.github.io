---
layout: post
title: stm32mp135faf7 PF13 can't remap to TIM2_CH1 PWM function
date: '2024-09-14 10:00:33 +0800'
categories: [linux, stm32mp135]
tags: [stm32mp135faf7]
description: 记录发现mp135资料有误的一次经历.
image:
  path: assets/images/3D_PCB_STM32MP135F_2024-05-07.png
  alt: my custom board.
---

## 问题现象
stm32mp135faf7配置PF13为TIM2_CH1 pwm无输出

## 源码版本
```text
源码下载：
buildroot:https://github.com/bootlin/buildroot.git 分支:st/2024.02.3
buildroot-external-st:https://github.com/bootlin/buildroot-external-st.git 分支:st/2024.02.3
编译编译：
$ cd buildroot/
$ make BR2_EXTERNAL=../buildroot-external-st st_stm32mp135f_dk_demo_defconfig
$ make
```
### 电路原理图
![Desktop View](assets/images/led.png){: width="972" height="589" }
![Desktop View](assets/images/pwm_io.png){: width="972" height="589" }
_Full screen width and center alignment_
### stm32mp135数据手册
![Desktop View](assets/images/ds_stm32.png){: width="972" height="589" }
_Full screen width and center alignment_

### 设备树配置如下
```text
tim2_pwm_pins_mx: tim2_pwm_mx-0 {
    pins {
        pinmux = <STM32_PINMUX('F', 13, AF1)>; /* TIM2_CH1 */
        bias-pull-up;
        drive-push-pull;
        slew-rate = <3>;
    };
};

tim2_pwm_sleep_pins_mx: tim2_pwm_sleep_mx-0 {
    pins {
        pinmux = <STM32_PINMUX('F', 13, ANALOG)>; /* TIM2_CH1 */
    };
};
&timers2{
    status = "okay";

    /* USER CODE BEGIN timers2 */
    /* USER CODE END timers2 */

    pwm{
        pinctrl-names = "default", "sleep";
        pinctrl-0 = <&tim2_pwm_pins_mx>;
        pinctrl-1 = <&tim2_pwm_sleep_pins_mx>;
        status = "okay";

        /* USER CODE BEGIN timers2_pwm */
        /* USER CODE END timers2_pwm */
    };
};
```
## pwm输出使能测试
代码编译烧录到板子上后，使用如下命令使能pwm输出
```text
cd /sys/class/pwm/pwmchip0
echo 0 > export
echo 100000000 > pwm0/period
echo 60000000 > pwm0/duty_cycle
echo 1 > pwm0/enable
```
pwm使用参考st官方wiki:<https://wiki.stmicroelectronics.cn/stm32mpu/wiki/PWM_overview>

## 验证PWM
- LED灯不亮
- 通过示波器测量PF13引脚无pwm波形
- 查看gpio工作状态显示PF13为ETR捕获模式，而不是pwm模式
![Desktop View](assets/images/driver3.png){: width="972" height="589" }
_Full screen width and center alignment_
同时内核驱动对PF13的支持也只有ETR，但PA5却明确显示支持CH1和ETR
![Desktop View](assets/images/driver1.png){: width="972" height="589" }
![Desktop View](assets/images/driver2.png){: width="972" height="589" }
_Full screen width and center alignment_

# 验证过程及方法
{: .mt-4 .mb-0 }
下面通过几方面来验证TIM2_CH1工作情况
### 验证TIM2是否工作
{: data-toc-skip='' .mt-4 .mb-0 }
使能4路pwm，验证TIM2是否能正常工作，结果是PA3 PB3 PB4对应的TIM2的三路channel有pwm输出，唯独PF13无pwm输出,说明TIM2能正常工作
```text
tim2_pwm_pins_mx: tim2_pwm_mx-0 {
    pins1 {
        pinmux = <STM32_PINMUX('A', 3, AF1)>, /* TIM2_CH4 */
                    <STM32_PINMUX('B', 3, AF1)>, /* TIM2_CH2 */
                    <STM32_PINMUX('B', 10, AF1)>; /* TIM2_CH3 */
        bias-disable;
        drive-push-pull;
        slew-rate = <0>;
    };
    pins2 {
        pinmux = <STM32_PINMUX('F', 13, AF1)>; /* TIM2_CH1 */
        bias-pull-up;
        drive-push-pull;
        slew-rate = <3>;
    };
};

tim2_pwm_sleep_pins_mx: tim2_pwm_sleep_mx-0 {
    pins {
        pinmux = <STM32_PINMUX('A', 3, ANALOG)>, /* TIM2_CH4 */
                    <STM32_PINMUX('F', 13, ANALOG)>, /* TIM2_CH1 */
                    <STM32_PINMUX('B', 3, ANALOG)>, /* TIM2_CH2 */
                    <STM32_PINMUX('B', 10, ANALOG)>; /* TIM2_CH3 */
    };
};
```
### 验证CH1能否输出到其它GPIO
{: data-toc-skip='' .mt-4 .mb-0 }
配置TIM2_CH1到PA5引脚，发现PA5能有pwm输出。led灯也能通过调试占空比改变亮度。说明TIM2_CH1本身功能正常，那只能怀疑是PF13引脚不支持pwm输出，可能文档有误
```text
tim2_pwm_pins_mx: tim2_pwm_mx-0 {
    pins {
        pinmux = <STM32_PINMUX('A', 5, AF1)>; /* TIM2_CH1 */
        bias-pull-up;
        drive-push-pull;
        slew-rate = <3>;
    };
};

tim2_pwm_sleep_pins_mx: tim2_pwm_sleep_mx-0 {
    pins {
        pinmux = <STM32_PINMUX('A', 5, ANALOG)>; /* TIM2_CH1 */
    };
};
```
### 裸机验证PF13输出PWM
{: data-toc-skip='' .mt-4 .mb-0 }
想着可能是内核驱动有问题，用裸机验证，虽然裸机代码确实有TIM2_CH1配置，但PF13还是无pwm输出，led灯不亮
![Desktop View](assets/images/bare_meta1.png){: width="972" height="589" }
![Desktop View](assets/images/bare_meta2.png){: width="972" height="589" }
_Full screen width and center alignment_

裸机开发建立工程参考:<https://blog.csdn.net/qq_14883963/article/details/135903186?spm=1001.2014.3001.5502>

### st论坛求助
{: data-toc-skip='' .mt-4 .mb-0 }
<https://community.st.com/t5/stm32-mpus-products/tim2-ch2-pwm-not-work/m-p/718198#M11888>
# 结论
stm32mp135的官方手册有误
正确配置如下:
- PA0, PA15, PD3, PG8 only TIM2_CH1
- PE2, PF13, PE15 only TIM2_ETR
- PA5 both TIM2_CH1, TIM2_ETR