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

## Overview of Pastry detection algorithm (Flow Chart)




![image](https://github.com/user-attachments/assets/f40bb396-51b3-4330-b3ed-b9339d621682)


