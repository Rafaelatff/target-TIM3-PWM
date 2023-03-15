# target-TIM3-PWM
Code generated during studies for course 'Mastering Microcontroller: Timers, PWM, CAN, Low Power(MCU2)'

## Coding a buzzer for a STM32H7xx

The buzzer works on a frequency of 2731 Hz. Let's code a PWM signal that has this frquency and 50% duty cycle. We selected PB1 pin that has available TIM3.

The struct to configure the timer 3 is presented next:

```c
/**
  * @brief  TIM Time Base Handle Structure definition
  */
#if (USE_HAL_TIM_REGISTER_CALLBACKS == 1)
typedef struct __TIM_HandleTypeDef
#else
typedef struct
#endif /* USE_HAL_TIM_REGISTER_CALLBACKS */
{
  TIM_TypeDef                        *Instance;         /*!< Register base address                             */
  TIM_Base_InitTypeDef               Init;              /*!< TIM Time Base required parameters                 */
  HAL_TIM_ActiveChannel              Channel;           /*!< Active channel                                    */
  DMA_HandleTypeDef                  *hdma[7];          /*!< DMA Handlers array
                                                             This array is accessed by a @ref DMA_Handle_index */
  HAL_LockTypeDef                    Lock;              /*!< Locking object                                    */
  __IO HAL_TIM_StateTypeDef          State;             /*!< TIM operation state                               */
  __IO HAL_TIM_ChannelStateTypeDef   ChannelState[6];   /*!< TIM channel operation state                       */
  __IO HAL_TIM_ChannelStateTypeDef   ChannelNState[4];  /*!< TIM complementary channel operation state         */
  __IO HAL_TIM_DMABurstStateTypeDef  DMABurstState;     /*!< DMA burst operation state                         */
}
```

The **Init** is another *typedef struct* that has a lot of parameters. Those can be seeing next:

```c
/**
  * @brief  TIM Time base Configuration Structure definition
  */
typedef struct
{
  uint32_t Prescaler;         /*!< Specifies the prescaler value used to divide the TIM clock.
                                   This parameter can be a number between Min_Data = 0x0000 and Max_Data = 0xFFFF */

  uint32_t CounterMode;       /*!< Specifies the counter mode.
                                   This parameter can be a value of @ref TIM_Counter_Mode */

  uint32_t Period;            /*!< Specifies the period value to be loaded into the active
                                   Auto-Reload Register at the next update event.
                                   This parameter can be a number between Min_Data = 0x0000 and Max_Data = 0xFFFF.  */

  uint32_t ClockDivision;     /*!< Specifies the clock division.
                                   This parameter can be a value of @ref TIM_ClockDivision */

  uint32_t RepetitionCounter;  /*!< Specifies the repetition counter value. Each time the RCR downcounter
                                    reaches zero, an update event is generated and counting restarts
                                    from the RCR value (N).
                                    This means in PWM mode that (N+1) corresponds to:
                                        - the number of PWM periods in edge-aligned mode
                                        - the number of half PWM period in center-aligned mode
                                     GP timers: this parameter must be a number between Min_Data = 0x00 and
                                     Max_Data = 0xFF.
                                     Advanced timers: this parameter must be a number between Min_Data = 0x0000 and
                                     Max_Data = 0xFFFF. */

  uint32_t AutoReloadPreload;  /*!< Specifies the auto-reload preload.
                                   This parameter can be a value of @ref TIM_AutoReloadPreload */
} TIM_Base_InitTypeDef;
```

The generated code, using STM32CubeMX for TIM 3 were:

```c
  htim3.Instance = TIM3;
  htim3.Init.Prescaler = 0;
  htim3.Init.CounterMode = TIM_COUNTERMODE_UP;
  htim3.Init.Period = 65535;
  htim3.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
  htim3.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
  if (HAL_TIM_PWM_Init(&htim3) != HAL_OK)
  {
    Error_Handler();
  }
```

So the code: (lesson 65 of the course explains how to configure)

* htim3.Instance = TIM3;  -> configure the hw to use timer 3
* htim3.Init.Prescaler = 0; -> configure the use of a pre-scaler 

The TIM 3 is connected on APB1 bus (according to STM32H743 datasheet).

![image](https://user-images.githubusercontent.com/58916022/225403175-6503d1e6-adf5-4477-a7c8-4ab86bf4d020.png)

And the clock frequencies according to Reference Manual are:

![image](https://user-images.githubusercontent.com/58916022/225404135-4a3bed7b-c8ec-4671-877a-ff3368a79d23.png)

Following the 'RCC clock configuration register (RCC_CFGR)' and 'RCC domain 2 clock configuration register (RCC_D2CFGR)', we have the bits 6:4 that configure the 'D2PPRE1[2:0]: D2 domain APB1 prescaler'.

![image](https://user-images.githubusercontent.com/58916022/225405269-f4c5e494-bc1f-491d-9c1e-06f1f7ed617a.png)

![image](https://user-images.githubusercontent.com/58916022/225405893-d91868e1-447c-4200-83a1-214058261c06.png)

Before this clock we have the HPRE prescaler that is responsable for divide the CPU clock by its values. CPU value = SYSCLK / D1CPRE prescaler.

![image](https://user-images.githubusercontent.com/58916022/225406981-2c9feb95-115d-41b5-986a-972331e1e637.png)

![image](https://user-images.githubusercontent.com/58916022/225407140-a5226113-f535-4eda-9649-0f8587e023de.png)

Then, the first prescaler that passes thought the SYSCLK, is the D1CPRE, shower next:

![image](https://user-images.githubusercontent.com/58916022/225407883-3dda4f56-a872-45fb-96fd-404c30562cc0.png)

![image](https://user-images.githubusercontent.com/58916022/225408005-e6d8b5e6-9390-4efc-8421-8fbb97a62403.png)

The prescaler value is used to slow down the TIM_CLK (TIM_CLK = 50 MHz, prescaler = 1, then TIM_CNT_CLK = TIM_CLK/ 1+ prescaler = 25 MHz).

* htim3.Init.CounterMode = TIM_COUNTERMODE_UP; -> only option in basic timers, 
