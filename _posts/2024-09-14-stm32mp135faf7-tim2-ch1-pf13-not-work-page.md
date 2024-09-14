---
layout: post
title: stm32mp135faf7 TIM2_CH1 on PF13 can not generate pwm
date: '2024-09-14 10:00:33 +0800'
categories: [linux, stm32mp135]
tags: [stm32mp135faf7]
description: a debug touris of stm32mp135 pwm.
image:
  path: assets/images/3D_PCB_STM32MP135F_2024-05-07.png
  alt: my custom board.
---

## 问题现象
stm32mp135faf7配置PF13为TIM2_CH1 pwm无输出
### 设备树配置如下

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
## 排查思路
# 配置TIM2的4路pwm，验证
{: .mt-4 .mb-0 }

## Links

<https://community.st.com/t5/stm32-mpus-products/tim2-ch2-pwm-not-work/m-p/718198#M11888>
