# Dual Channel Output Compare with STM32 (TIM2)

This project demonstrates how to use **two channels of TIM2** on an STM32 microcontroller to generate **two output signals with different frequencies** using **Output Compare (OC) mode**.  

---

## Project Goal
The main purpose of this project is **educational practice**. It helps developers understand and experiment with:  
- How **Output Compare (OC) mode** works in STM32 timers.  
- How to generate **two different square wave signals** using **two channels of the same timer**.  
- How **Prescaler**, **ARR (Auto-Reload Register)**, and **CCR (Compare Register)** values interact to define **frequency and phase**.  
- How to use **interrupts** to update CCR values dynamically and change the phase of the waveform.  

---

## Project Description
We configure **TIM2** with two active channels:
- **Channel 1**: Toggles at a frequency defined by `compare register (CCR1)`  
- **Channel 2**: Toggles at a frequency defined by `compare register (CCR2)`  

The **timer base frequency** is determined by:
```

Timer_Freq = (System_Clock / Prescaler)

```
and the **output waveform frequency** depends on:
```

Output_Freq = Timer_Freq / (2 * (ARR + 1))

````
while the **phase** of the signal can be shifted using the **Compare Register (CCR)** values.

In toggle mode, the duty cycle is always **50%**, and only the phase changes when CCR is updated.

---

## Configuration
- **System Clock**: 8 MHz (HSI-based, PLL enabled)  
- **TIM2 Prescaler**: `1000 - 1`  
- **ARR (Auto-Reload Register)**: `16000 - 1`  
- **Channel 1 CCR**: `4000 - 1`  
- **Channel 2 CCR**: `8000 - 1`  
- **Mode**: Output Compare (Toggle)  
- **Interrupts**: Enabled for both channels  

---

## Code Explanation
### Callback Function
Every time the counter matches the compare register value, the interrupt triggers `HAL_TIM_OC_DelayElapsedCallback`.  
```c
void HAL_TIM_OC_DelayElapsedCallback(TIM_HandleTypeDef *htim)
{
	if(htim->Instance == TIM2){
		if(htim->Channel== HAL_TIM_ACTIVE_CHANNEL_1){
			uint16_t comp = __HAL_TIM_GetCompare(htim,TIM_CHANNEL_1);
			__HAL_TIM_SetCompare(htim,TIM_CHANNEL_1,(comp<16000-1)?(comp+=4000):(4000-1));
		}
		else if(htim->Channel== HAL_TIM_ACTIVE_CHANNEL_2){
			uint16_t comp = __HAL_TIM_GetCompare(htim,TIM_CHANNEL_2);
			__HAL_TIM_SetCompare(htim,TIM_CHANNEL_2,(comp<16000-1)?(comp+=8000):(8000-1));
		}
	}
}
````

* **Channel 1** increments CCR by `4000` → defines frequency `f1`.
* **Channel 2** increments CCR by `8000` → defines frequency `f2`.
* Both signals are toggled with a **50% duty cycle**.

### Timer Initialization

```c
htim2.Instance = TIM2;
htim2.Init.Prescaler = 1000-1;
htim2.Init.Period = 16000-1;
HAL_TIM_OC_Init(&htim2);

sConfigOC.OCMode = TIM_OCMODE_TOGGLE;
sConfigOC.Pulse = 4000-1;
HAL_TIM_OC_ConfigChannel(&htim2, &sConfigOC, TIM_CHANNEL_1);

sConfigOC.Pulse = 8000-1;
HAL_TIM_OC_ConfigChannel(&htim2, &sConfigOC, TIM_CHANNEL_2);
```

