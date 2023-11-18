# v_eep2
 Simple >>Virtual EEPROM<< library for STM32F0 MCUs.
Uses STM32 HAL.
 Library uses two FLASH pages to store one(!) data
structure in non-volatile memory.
Library allows to use only one data structure in the
project. It stores 2 copies of the data, and manages
data corruption. Each copy is in different FLASH Page, 
thus it is more relaiable.
You can store up to [FLASH PAGE SIZE]-4B (for CRC).




[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)



## Usage/Examples

Remember to change following defines in v_eep2.h to FLASH pages of your selection:

```c
#define V_EEP_PAGE1	0x0803F000 	//Page 126, size: 2K
#define V_EEP_PAGE2	0x0803F800	//Page 127, size: 2K
```

In Cube MX activate CRC and (optionaly) IWDG.

### How to find a proper FLASH page address?
Choose last page of your FLASH. Please refer to the MCU documentation, or you can also
look on the ST-Link Utility -> Target -> Erase sectors to determine the
last page address.


```c

/*
 * Define here starting addresses of the last FLASH pages, that will be used
 * for data storage. Please refer to the MCU documentation, or you can also
 * look on the ST-Link Utility -> Target -> Erase sectors to determine the
 * last page address.
 * \image html memory_map.jpg "Example memory map in ST-Link Utility"
*/
#include "v_eep.h"

struct myImportantData {
    int someParameter;
    int someArray[10];
    uint32_t timer;
} myImportantData;

main(){
	//set IWDG timeout long enough for 2 cycles of FLASH erase
	HAL_IWDG_Refresh(&hiwdg);
	v_eep_read_result_t res = v_eep_read_verified(&myImportantData, sizeof(myImportantData), &hcrc);
	HAL_IWDG_Refresh(&hiwdg);
	if( res == V_EEP_RESULT_OK ){

		printf("Data restored from Virtual EEPROM!\r\n");
		
	}else if( res == V_EEP_RESULT_1STOK ){

		printf("Data restored from 1st copy of Virtual EEPROM!\r\n");
		
	}else if( res == V_EEP_RESULT_2NDOK ){

		printf("Data restored from 2nd copy of Virtual EEPROM!\r\n");
		
	}else if( res == V_EEP_RESULT_FAIL ){

		printf("Both copies of the data corrupted! Initializing with default values.\r\n");

	}
	HAL_IWDG_Refresh(&hiwdg);
	///...

	myImportantData.someParameter += 1;

	v_eep_write(&myImportantData, sizeof(myImportantData), &hcrc);
	HAL_IWDG_Refresh(&hiwdg);
	printf("someParameter = %d\r\n", myImportantData.someParameter);
	
}

```

