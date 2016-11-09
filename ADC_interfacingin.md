#include "stdafx.h"




#include <windows.h>
#include <stdio.h>
#include "UsbI2cIo.h"


void display_digit(HANDLE hDevInstance, int num, int digit);
void display_number(HANDLE hDevInstance, int num);
long adc_read(HANDLE hDevInstanc);
long adc_value = 0;


void delay() { //delay loop 
    int t = 0;
    while (t < 999999) {
        t++;
    }
}


void __cdecl main(void) {


    HANDLE hDevInstance = INVALID_HANDLE_VALUE; // device instance handle
                                                // attempt to open device UsbI2cIo0 (instance 0 of the device with symbolic name UsbI2cIo)
    hDevInstance = DAPI_OpenDeviceInstance("UsbI2cIo", 0);


    if (hDevInstance != INVALID_HANDLE_VALUE) {
        // success - device instance handle opened
        printf("Opened device UsbI2cIo0\n");


        DAPI_ConfigIoPorts(hDevInstance, 0x00000011); // 0 = output, 1 = input
        printf("Configured all I/O ports as outputs\n");




        for (;;) {
            DAPI_WriteIoPorts(hDevInstance, 0x0000FD00, 0xFFFFFFFF); //
            adc_value = adc_read(hDevInstance);
            delay;
            DAPI_WriteIoPorts(hDevInstance, 0x000F0000, 0xFFFFFFFF); // testing if 7-segment works
        }
    }


    else {
        printf("Failed to open a handle to device UsbI2cIo0\n");
    }


    if (hDevInstance != INVALID_HANDLE_VALUE) {
        // close handle to device instance
        DAPI_CloseDeviceInstance(hDevInstance);
    }
    system("pause");
}


long adc_read(HANDLE hDevInstance) {


    long input;
    DAPI_ReadIoPorts(hDevInstance, &input);


    return input >> 16;
    return input;
}


int conversion(int num) {
    return(num * 5000) / 255; // 5000 is the value that converts 8 digits to 8 digit output on the ADC , 255 is the possible combinations of the 8bit 
}


void display_number(HANDLE hDevInstance, int num) {


    display_digit(hDevInstance, num / 1000, 0x8); //display digits on the first value of the 7-segment
    display_digit(hDevInstance, num / 100, 0x4);  //display digits on the second value of the 7-segment
    display_digit(hDevInstance, num / 10, 0x2);   //display digits on the third value of the 7-segment
    display_digit(hDevInstance, num, 0x1);          // display digits on the 
}


void display_digit(HANDLE hDevInstance, int num, int digit) {


    unsigned int output; //set output type 


    switch (num) {    //use to switch values to another value 
    case 0:
        output = 0x10; // when segment shows 0
        break;
    case 1:
        output = 0xBB;// 7-segment value as 1
        break;
    case 2:
        output = 0x26; //7-segment value as 2
        break;
    case 3:
        output = 0x2A; // when segment shows 3
        break;
    case 4:
        output = 0x8B; //7-segment value as 4
        break;
    case 5:
        output = 0x4A; //7-segment value as 5
        break;
    case 6:
        output = 0x40;//7 segment value as 6
        break;
    case 7:
        output = 0x3B; //7-segment value as 7
        break;
    case 8:
        output = 0x00; //7-segment value as 8
        break;
    case 9:
        output = 0x08; //7-segment value as 9
        break;
    default:
        output = 0x10; //go back to 0
        break;
    }
}


