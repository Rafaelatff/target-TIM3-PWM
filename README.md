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

## Configuring the bus timer clock (APB1 timer clock)

So the code: (lesson 65 of the course explains how to configure)

* htim3.Instance = TIM3;  -> configure the hw to use timer 3
* htim3.Init.Prescaler = 0; -> configure the use of a pre-scaler 

The TIM 3 is connected on APB1 bus (according to STM32H743 datasheet).

![image](https://user-images.githubusercontent.com/58916022/225403175-6503d1e6-adf5-4477-a7c8-4ab86bf4d020.png)

And the clock frequencies according to Reference Manual are:

![image](https://user-images.githubusercontent.com/58916022/225404135-4a3bed7b-c8ec-4671-877a-ff3368a79d23.png)

The clock three (that comes with STM32CubeMX) is showed next:

![image](https://user-images.githubusercontent.com/58916022/225408436-d24b70a6-2000-4ce6-a767-4b74d1e46800.png)

Then, the first prescaler that passes thought the SYSCLK, is the D1CPRE, shower next:

![image](https://user-images.githubusercontent.com/58916022/225407883-3dda4f56-a872-45fb-96fd-404c30562cc0.png)

![image](https://user-images.githubusercontent.com/58916022/225408005-e6d8b5e6-9390-4efc-8421-8fbb97a62403.png)

After this clock we have the HPRE prescaler that is responsable for divide the CPU clock by its values. CPU value = SYSCLK / D1CPRE prescaler.

![image](https://user-images.githubusercontent.com/58916022/225406981-2c9feb95-115d-41b5-986a-972331e1e637.png)

![image](https://user-images.githubusercontent.com/58916022/225407140-a5226113-f535-4eda-9649-0f8587e023de.png)

Following the 'RCC clock configuration register (RCC_CFGR)' and 'RCC domain 2 clock configuration register (RCC_D2CFGR)', we have the bits 6:4 that configure the 'D2PPRE1[2:0]: D2 domain APB1 prescaler'.

![image](https://user-images.githubusercontent.com/58916022/225405269-f4c5e494-bc1f-491d-9c1e-06f1f7ed617a.png)

![image](https://user-images.githubusercontent.com/58916022/225405893-d91868e1-447c-4200-83a1-214058261c06.png)

PCLK1 is equal to the APB1 bus. If we debug and ask for the PCL1, as shown next:

```c
 printf("PCLK1: %ld\r\n", HAL_RCC_GetPCLK1Freq());
```

We will have as results:

![image](https://user-images.githubusercontent.com/58916022/225409749-716d8fbb-b4b5-45d3-8fc3-2c148fd992a5.png)

But timer uses a multiplier, to multiply the APB1 frequency.

![image](https://user-images.githubusercontent.com/58916022/225410126-e396e770-344b-4871-88ab-08ef6ada80ae.png)

The **multiplier value** is according to **D2PPRE1 value**, and can be 1 or 2 only (is 1, only if D2PPRE1 is also 1).

## Configuring the period for timer (and PWM eventually)

After configuring the APB1 Timer clock, we need to configure the presscaler that comes with the timer clock itself (64 MHz). Acording to the following image (Reference manual), TIMxCLK from RCC or CK_INT = CK_PSC. Then the prescaler PSC comes and its output (CNT_CLK ou counter clock) = TIMx_CLK / (prescaler +1).

![image](https://user-images.githubusercontent.com/58916022/225420040-68452c80-7d32-4733-9c30-e25c14935063.png)

The prescaler value is used to slow down the TIM_CLK (TIM_CLK = 50 MHz, prescaler = 1, then TIM_CNT_CLK = TIM_CLK/ 1+ prescaler = 25 MHz). Once we need a 2731 Hz, that is a lot smaller than 64 MHz we can use a prescaler of 9999, just to have a smaller frequency value to our clock timer. We will have 64 M / 10000 = 6400 Hz. Or we can say it is 0.0001562 seconds (1.562 us).

* htim3.Init.Prescaler = 9999;  
* htim3.Init.CounterMode = TIM_COUNTERMODE_UP; -> only option in basic timers, in our case its okay to use this option.
* htim3.Init.Period = 1;

Now just a small reminder:

```c
  uint32_t Period;            /*!< Specifies the period value to be loaded into the active
                                   Auto-Reload Register at the next update event.
                                   This parameter can be a number between Min_Data = 0x0000 and Max_Data = 0xFFFF.  */
```

Period is the value of this block:

![image](https://user-images.githubusercontent.com/58916022/225423314-fbe89917-33d8-4398-b5d3-ff107ac3a8fb.png)

Period value is copied to the ARR - Auto Reload Register, to generate the clock cycles. Both are 16 bits (value between 0x0000 to 0xFFFF). Both values cannot be 0, or the timer won't start.

* TIM_CLK = 64 MHZ
* CNT_CLK = 6400 Hz = 0.0001562 seconds (3.5 us)
* We need 2731 Hz = 0.0003661 seconds.
* So, period will be 0.0003661/0.0001562 = 2.34379 (lets use 2 -> 3201 Hz).

(NOTE: period cannot be 1. pulse variable is the % of the value.)

(TIP: create a excell sheet to quickly calculate this)

This way, we have configured the **HAL_TIM_PWM_Init(&htim3)**.

## PWM configurations

Now lets configure the **HAL_TIM_PWM_ConfigChannel**. For that, we need to configure the *TIM_OC_InitTypeDef* sConfigOC (output compare). Lesson 94 explains how it works.

```c
/**
  * @brief  TIM Output Compare Configuration Structure definition
  */
typedef struct
{
  uint32_t OCMode;        /*!< Specifies the TIM mode.
                               This parameter can be a value of @ref TIM_Output_Compare_and_PWM_modes */

  uint32_t Pulse;         /*!< Specifies the pulse value to be loaded into the Capture Compare Register.
                               This parameter can be a number between Min_Data = 0x0000 and Max_Data = 0xFFFF */

  uint32_t OCPolarity;    /*!< Specifies the output polarity.
                               This parameter can be a value of @ref TIM_Output_Compare_Polarity */

  uint32_t OCNPolarity;   /*!< Specifies the complementary output polarity.
                               This parameter can be a value of @ref TIM_Output_Compare_N_Polarity
                               @note This parameter is valid only for timer instances supporting break feature. */

  uint32_t OCFastMode;    /*!< Specifies the Fast mode state.
                               This parameter can be a value of @ref TIM_Output_Fast_State
                               @note This parameter is valid only in PWM1 and PWM2 mode. */


  uint32_t OCIdleState;   /*!< Specifies the TIM Output Compare pin state during Idle state.
                               This parameter can be a value of @ref TIM_Output_Compare_Idle_State
                               @note This parameter is valid only for timer instances supporting break feature. */

  uint32_t OCNIdleState;  /*!< Specifies the TIM Output Compare pin state during Idle state.
                               This parameter can be a value of @ref TIM_Output_Compare_N_Idle_State
                               @note This parameter is valid only for timer instances supporting break feature. */
} TIM_OC_InitTypeDef;
```

Next the 3 main parameters.

* sConfigOC.OCMode = TIM_OCMODE_PWM1;
* sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;

Polarity high means pulse calcule will be based on high percentage of duty cycle.

* sConfigOC.Pulse = 1;

The duty cycle is configured by configuring the pulse value (that depends on period and ARR value).

* to use 40% duty cycle
* pulse (CCR1) = ARR * (40/100)
* pulse (CCR1) = period * %
* example: sConfigOC.Pulse = (htim3.Init.Period * 40); to use 40% duty cycle.

And to finish, inside the main() source file, we pass the parameters and call TIM_CHANNEL_4.

```c
if (HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_4)!= HAL_OK) Error_Handler();
```
