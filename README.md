# "E.A.S.Y (EveBot Automated Solution, Why?)" Software Documentation


## "(EveBot Automated Solution, Why?)" Description
The system consist of **key components reflecting functionalities** that is the 
  + Pixy Camera for positional guidance
  + PrintHead/X Axis Carriage for X Axis Movement
  + Conveyer for Y Axis Movement
  + Guiding Rod attached to printhead for Z Axis Movement
  + Keypad for program control

The software integrates all of the components, with the keypad being the control panel, X and Y axis fully automated with PixyCam as the main driver for positional movement and manual/human control for Z Axis.


## Pixy Camera For Positional Guidance

The `Pixy Camera` inteprets the conveyer belt based on image below. These objects are pastries, with `colour signature` boxes on them. ***Based on the axes, towards right x value increases. As you go
down, y value increases.***

![image](https://github.com/user-attachments/assets/aed40e77-0457-40ac-9778-4c36f20d2207)




The pixy camera has two lines to create a `row` of objects along the `X-axis`.

![image](https://github.com/user-attachments/assets/625fcfe4-b290-42a9-aa89-1424a91e577e)


### Row Object

#### Creating a `Row` Reference Point

To guide X and Y Axis to move and print on which pastries, the program creates a `Row`, consist of `x coordinates` of pastries along the same row, a y coordinate `coordRow_Y` that serves as **a reference point to bring the `Row` to the `PRINTLINE`** and the number of pastries/objects in the `Row`.

```
typedef struct __row{
	uint16_t coordRow_Y;
	uint16_t numberOfRowObjects;
	uint16_t coordXArray[10];
}Row;
```

**The program creates a reference point for a `Row`, as described in the code snippet below.**

The bottommost pastry i.e. pastry closest to the `PRINTLINE` is the reference point for a `Row`.

```
	for(int j = 0; j<numberOfBlocks; ++j)
	{
		//find the lowest centre point of an object within a "row" AND ONLY IF
		//The bottom width of the object is before the EVENT HORIZON
		//the lowest centre point of the object will be the main reference for a "row"
		
		if(pastry[j].coordY > rowval && pastry[j].bottomY<= EVENT_HORIZON) // EVENT_HORIZON is PRINTLINE - 10 pixels (Before PRINTLINE, hence subtract)
		{
			rowval = pastry[j].coordY;
		}

	}
```




The program first finds the pastry closes to the `PRINTLINE` i.e. bottommost pastry along the `Y Axis` by looking at **centrepoints** of these pastries. Finding the bottommost pastry among other pastries require `rowval` for comparison. The bottommost pastry is selected at the end of the **for loop**. This is the program **first condition**.

The **second condition** dictates that the pastry selected as the reference for `Row` must have its bottom width **before** the `PRINT_LINE` + 10 pixels line.

The images below descibes cases where `Row` is created based on conditions.

![image](https://github.com/user-attachments/assets/5df17082-ff4e-45d8-a77f-4613ccdc6bd4)


The image above describes a row being created because it **meets the two conditions mentioned**.

![image](https://github.com/user-attachments/assets/787e62d6-583e-4ee2-aa34-df8b97222f40)


The image above shows that **no row is created** because it **does not meet the second condition**.

### Adding Pastries to the `Row` Object

Once `Row` reference point is selected i.e. closest to `PRINT_LINE`, the program now adds other pastries along the `Y axis` within a buffer set.

Taking `Row.coordRow_Y`, which was set as a reference point along the Y axis i.e. conveyer, the program looks for pastries with centre point within `Row.coordRow_Y` and `Row.coordRow_Y - 10`. The snippet
describes this pastry selection for the `Row` object.

```
	for(int i = 0; i< numberOfBlocks; ++i)//loop for the no objects
	{
		//since the lowest centrepoint is the reference for the row,
		//we account for objects within the gap
		//before the lowest centrepoint
		if(pastry[i].coordY >= (rowval - 10) && pastry[i].coordY <= rowval){
			row.coordXArray[i] = pastry[i].coordX;
			row.numberOfRowObjects++;
		}
	}
```

The following image describes that two crossaints are selected because their centrepoints along the y axis is between the `Row.coordRow_Y` and `Row.coordRow_Y - 10`.

![image](https://github.com/user-attachments/assets/1a854c1c-a923-4bcc-acf2-82e6888d42e6)


Hence, a row is created with the `Row.coordXArray[]` **filled with the centrepoints of pastries along the x axis.**

### Sorting the `Row.coordXArray[]`

The row of x coordinates is sorted from smallest (closest to the origin or printhead) to the largest. This ensures that the printhead goes in proper sequence from left to right.

```
bubbleSort(row.coordXArray, 10);
```

## X Axis Print Head Movement

The X Axis Movement has the following **key** functions to guide the printhead to locations where the pastry is placed based on pixy camera.
    
+ `printHeadMovementRoutine_AUTO()`
+ `printHeadMovementRoutine_Manual()`

Both of these functions share the same sequence, except that `printHeadMovementRoutine_Manual()` does not have `pressEveBot()`, therefore requires a human to press within a 3 seconds window given.

These two functions execute a logical sequence, calling helper functions shown in the following diagram below:

+ `xAxisMovement((row.coordXArray[i]- datum) * PIXY_RATIO)`


The `xAxisMovement((row.coordXArray[i]- datum) * PIXY_RATIO)` moves the printhead to the centrepoints of pastries, starting with the first pastry. The `datum` is a preset value that offsets the printhead from the centrepoint of a pastry. `PIXY_RATIO` is a ratio used to convrtt the **number of pixels** from `(row.coordXArray[i]- datum)` into distance(mm).


The diagram below illustrates this function.

![image](https://github.com/user-attachments/assets/7c73d173-be63-474d-b112-0becbcb0d889)



+ `iniDelay()`

Once arrived at the location within an offset from the centrepoint of the pastry, the program commences `iniDelay()`. `iniDelay()` covers the distance gap EveBot is required to roll before actual printing commences. The distance gap was measured from the EveBot ruler illustrated below.

![image](https://github.com/user-attachments/assets/a65f1a53-ebe0-48de-bbd9-2b8eccbe0d74)


+ `xAxisMovement(PRINTSIZE)`

The printhead then moves according to the PRINTSIZE specified by the user in via the **keypad interface.** The default value for `PRINTSIZE` is 43. 

![image](https://github.com/user-attachments/assets/ce2e664e-ba24-44e2-97bb-c1ae75bb352b)


+ `backToHomePosition()`

This function moves the printhead back to the HOME position i.e. until it hits the limit switch at HOME. Within this function, the program sends **HIGH** and **LOW** pulses until the limit switch toggles. The following diagram illustrates this:

![image](https://github.com/user-attachments/assets/93310df3-40e2-4cf7-aeac-56de80c900d2)


+ `pressEveBot()`

This function sends PWM signals generated by `TIM3` peripheral to the servo motor attached to our mechanical button presser as shown below. It retracts the pinion presser up, before pressing down and goes back up again.



### More information about the Helper Functions

+ `xAxisMovement(int distance)`

The function converts the arguement that is in mm the program assumes into **steps** for the stepper motor. The **steps** are for loop iterations, which sends
**HIGH** and **LOW** pulses  in **alternating and fixed microseconds interval** to the motor driver. This routine is similar to conveyer stepper motor and z axis stepper motor
movement, exept with differing values. The following code below showcases this:

```
void xAxisMovement(int distance){
    int dist = distCal(distance);
    int stepDelay = 1900;
    for(int x =0; x<dist;++x)
    {
		HAL_GPIO_TogglePin(PRINTHEAD_STEP_PORT, PRINTHEAD_STEP_PIN);
		microDelay(stepDelay);
    }
}
```
+ `pressEveBot()`

The STM32F103RB is configured to generate PWM signals at 50HZ. 


```
  __HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_3, RETRACT_PEN_UP);
  HAL_Delay(500);
  __HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_3, EXTEND_PEN_DOWN);
  HAL_Delay(500);
  __HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_3, RETRACT_PEN_UP);
  HAL_Delay(500);
 
```
The `__HAL_TIM_SET_COMPARE()` describes `TIM3` being used, while the third parameter
is the **duty cycle** you would like to set. Since the counter period is set to 1000 on the stm, `RETRACT_PEN_UP` and `EXTEND_PEN_DOWN` are given values of **200** and **336** respectively. That means divided by the counter period, **duty cycles are 20% and 36% respectively**.


## Y Axis Print Head Movement
The Y Axis Movement has the following **key** functions to guide and move the conveyer belt to move the tray of pastries along the Y axis.
+ `conveyerMovement(int z)`

Similar to `xAxisMovement(int distance)`, the function moves the conveyer stepper motor, as specified by int z (distance in mm).

+ `conveyerMovePastry()`

This function additional computes distance required to move the **reference point** of a row to the `PRINTLINE`.

`row.coordRow_Y` is the **difference in distance in pixels** between the `PRINTLINE` and the **y coordinate** of the `row` reference point i.e. lowest pastry in the row. The following code snipper AND a visual diagram illustrates this.

```
void conveyerMovePastry(){
	int distanceToTravel = row.coordRow_Y * PIXY_RATIO;
	conveyerMovement(distanceToTravel);

}
```

![image](https://github.com/user-attachments/assets/9385b23c-a41c-487c-a324-9f9cae496d1e)



## Z Axis Print Head Movement
The Z Axis Movement moves the print carriage vertically (upwards or downwards). The diagram below shows where the z axis stepper motor and guiding rod is at:

+ `void moveZAxisUp()`
+ `void moveZAxisDown()`

```
void moveZAxisUp(){
    int conveyerStepperDelay = 2000;
    GPIO_PinState zAxisDirectionStatus = HAL_GPIO_ReadPin(GPIOC, DIR_Z_AXIS_Pin);
    if(zAxisDirectionStatus == GPIO_PIN_RESET){
    	HAL_GPIO_TogglePin(GPIOC, DIR_Z_AXIS_Pin);
    	HAL_Delay(1);
    }
	HAL_GPIO_TogglePin(GPIOC, STEP_Z_AXIS_Pin);
	microDelay(conveyerStepperDelay);
}
```

+ `void moveZAxisDown()`
```
void moveZAxisDown(){
    int conveyerStepperDelay = 2000;
    GPIO_PinState zAxisDirectionStatus = HAL_GPIO_ReadPin(GPIOC, DIR_Z_AXIS_Pin);
    if(zAxisDirectionStatus == GPIO_PIN_SET){
    	HAL_GPIO_TogglePin(GPIOC, DIR_Z_AXIS_Pin);
    	HAL_Delay(1);
    }
	HAL_GPIO_TogglePin(GPIOC, STEP_Z_AXIS_Pin);
	microDelay(conveyerStepperDelay);
}

```

## Final Overview of the X Axis and Y Axis PrintHead and Conveyer Movement Routine (Flow Chart)

![image](https://github.com/user-attachments/assets/07cf07c0-ebe3-405b-bb9b-d552e8d4f683)



![image](https://github.com/user-attachments/assets/f40bb396-51b3-4330-b3ed-b9339d621682)


