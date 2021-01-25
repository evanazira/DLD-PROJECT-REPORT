#include <Keypad.h>
#include <Servo.h>
#include <EEPROM.h>
#include <LiquidCrystal.h>
const int LED1 = 11;
const int LED2 = 12;
int const numRows= 4;         
int const numCols= 4;      

char keymap[numRows][numCols]= 
{
{'1', '2', '3', 'A'}, 
{'4', '5', '6', 'B'}, 
{'7', '8', '9', 'C'},
{'*', '0', '#', 'D'}
};

char keypressed;                 
char password[]= {'1','2','3','4'};  


LiquidCrystal lcd(A0, A1, A2, A3, A4, A5);
byte rowPins[numRows] = {9,8,7,6}; 
byte colPins[numCols]= {5,4,3,2}; 

char check1[sizeof(password)];  
char check2[sizeof(password)];  
short a=0,i=0,s=0,j=0;          
Keypad myKeypad= Keypad(makeKeymap(keymap), rowPins, colPins, numRows, numCols);
Servo myservo;

void setup()
{
  lcd.begin (16,2);
  lcd.print("Standby"); //when no key is pressed
  myservo.attach(10);
  myservo.write(0); 
  pinMode (LED1,OUTPUT);
  pinMode (LED2,OUTPUT);
  pinMode (13,OUTPUT);
}

void loop()
{
  keypressed = myKeypad.getKey();
  if(keypressed == 'A')// A to open the lock keypad
  {
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Enter code");            //Message to show
    Readpassword();                          //Getting code function
    
    if(a==sizeof(password)) //The ReadCode function assign a value to a (it's correct when it has the size of the code array)
    {
      OpenDoor(); //Open lock function if code is correct
      digitalWrite(LED1, HIGH);
      tone (13,550,1000);
      delay(2000);
      CloseDoor();
      digitalWrite(LED1, LOW);
      
    }
    
    else
    {
      lcd.clear();
      lcd.print("Wrong"); //Message to print when the code is wrong
      digitalWrite(LED2, HIGH);
      tone (13,450,1000);
      delay(2000);
      digitalWrite(LED2, LOW);
    
    }
    
    delay(2000);
    lcd.clear();
    lcd.print("Standby");             //Return to standby mode it's the message do display when waiting
  }
  
  else if(keypressed == 'B') //'B' is to change the password
  {
    CloseDoor();
    Changepassword();
    lcd.clear();
    lcd.print("Standby");//When done it returns to standby mode
  }
  else if(keypressed == 'C')
  {
    lcd.clear();
    CloseDoor();
    lcd.clear();
    lcd.print("Standby");
  }
}

void Readpassword()
{                  //Getting code sequence
       i=0;                      //All variables set to 0
       a=0;
       j=0; 
                                     
     while(keypressed != '*'){                                     //The user press A to confirm the code otherwise he can keep typing
           keypressed = myKeypad.getKey();                         
             if(keypressed != NO_KEY && keypressed != '*' ){       //If the char typed isn't A and neither "nothing"
              lcd.setCursor(j,1);                                  //This to write "*" on the LCD whenever a key is pressed it's position is controlled by j
              lcd.print("*");
              j++;
            if(keypressed == password[i]&& i<sizeof(password)){            //if the char typed is correct a and i increments to verify the next caracter
                 a++;                                              
                 i++;
                 }
            else
                a--;                                               //if the character typed is wrong a decrements and cannot equal the size of code []
            }
            }
    keypressed = NO_KEY;

}


void Changepassword()
{                      //Change code sequence
  lcd.clear();
  lcd.print("Changing code");
  lcd.clear();
  lcd.print("Enter old code");
  Readpassword();                      //verify the old code first so you can change it
      
  if(a==sizeof(password))//again verifying the a value
  {      
    lcd.clear();
    lcd.print("Changing code");
    GetNewCode1();            //Get the new code
    GetNewCode2();            //Get the new code again to confirm it
    s=0;
    
    for(i=0 ; i<sizeof(password) ; i++)//Compare codes in array 1 and array 2 from two previous functions
    {
      if(check1[i]==check2[i])
        s++;                                //again this how we verifiy, increment s whenever codes are matching
    }
    if(s==sizeof(password))//Correct is always the size of the array
    {
      for(i=0 ; i<sizeof(password) ; i++)
      {
        password[i]=check2[i];         //the code array now receives the new code
        EEPROM.put(i, password[i]);        //And stores it in the EEPROM
      }
      lcd.clear();
      lcd.print("Password Changed");
      digitalWrite(LED1, HIGH);
      tone (13,550,1000);
      delay(500);
      digitalWrite(LED1, LOW);
    }
    else
    {                         //In case the new codes aren't matching
      lcd.clear();
      lcd.print("Password are not");
      lcd.setCursor(0,1);
      lcd.print("matching !!");
      digitalWrite(LED2, HIGH);
      tone (13,450,1000);
      delay(500);
      digitalWrite(LED2, LOW);
    }
  }
  else
  {                     //In case the old code is wrong you can't change it
    lcd.clear();
    lcd.print("Wrong");
    delay(1000);
  }
}

void GetNewCode1(){                      
  i=0;
  j=0;
  lcd.clear();
  lcd.print("Enter new code");   //tell the user to enter the new code and press A
  lcd.setCursor(0,1);
  lcd.print("and press *");
  delay(2000);
  lcd.clear();
  lcd.setCursor(0,1);
  lcd.print("and press *");     //Press A keep showing while the top row print ***
             
         while(keypressed != '*'){            //* to confirm and quits the loop
             keypressed = myKeypad.getKey();
               if(keypressed != NO_KEY && keypressed != '*' ){
                lcd.setCursor(j,0);
                lcd.print("*");               //On the new code you can show * as I did or change it to keypressed to show the keys
                check1[i]=keypressed;     //Store caracters in the array
                i++;
                j++;                    
                }
                }
keypressed = NO_KEY;
}

void GetNewCode2(){                         //This is exactly like the GetNewCode1 function but this time the code is stored in another array
  i=0;
  j=0;
  
  lcd.clear();
  lcd.print("Confirm password");
  lcd.setCursor(0,1);
  lcd.print("and press *");
  delay(3000);
  lcd.clear();
  lcd.setCursor(0,1);
  lcd.print("and press *");
  while(keypressed != '*')
  {
    keypressed = myKeypad.getKey();
    if(keypressed != NO_KEY && keypressed != '*' )
    {
      lcd.setCursor(j,0);
      lcd.print("*");
      check2[i]=keypressed;
      i++;
      j++;
    }
  }
keypressed = NO_KEY;
}

void OpenDoor(){             //Lock opening function open for 3s
  lcd.clear();
  lcd.print("Welcome");
  myservo.write(90);
  }

void CloseDoor()
{
  lcd.clear();
  lcd.print("Door is Locked!");
  myservo.write(0);
}
