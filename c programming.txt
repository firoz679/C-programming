#include<conio.h>
#include<stdio.h>
#include<graphics.h>
#include<dos.h>
/*
 *************************
 * Programmer :	  Abhishek Hingnikar                                       *
 * AIM        :   Mobile Shop Management System in C   			   *  
 *************************
*/

#define SCR_H 47 // Re-define as per requirement.

unsigned int _refId = 0;
FILE *f;

typedef struct{
	unsigned int refId;
	char name[30];
	char company[30];
	char IMEI[30];
	long float price;
}mobile;
// Prints a nice boundry and clears the screen

void prnt_scr() {

	int i = 0 ;
	clrscr();
	printf("################################################################################");

	for(i = 0; i < SCR_H; i++) {
		printf("#                                                                              #");
	}

	printf("################################################################################");
}

mobile * new_m() {
	mobile * m = (mobile *) malloc( sizeof( mobile ));  // wow even this aint no working.!
	float pr;
	prnt_scr();
	gotoxy(32,2);
	fflush(stdin);
	m->refId = _refId++;
	cprintf("Enter Phone Details");

	gotoxy(24,6);
	if( m->name == NULL ){
		textcolor(RED);
		clrscr();
		cprintf("Memory Leak");

		getch();
		exit(1);
	}
	cprintf("Model          - ");
	textbackground( 2 );
	gets( m->name );
	textbackground( BLACK );

	gotoxy( 24, 8);
	cprintf("Manufacturer   - ");
	gets( m->company );

	gotoxy( 24, 10);
	cprintf("IMEI           - ");
	gets(m->IMEI);

	gotoxy(24,12);
	cprintf("Price          - ");
	fflush(stdin); // flush or this will be a nada
	scanf("%f",&pr);
	//cprintf("Price Aqquired -",pr);
	m->price = pr;
	fseek(f,0,SEEK_END); // write a new at the end of file.

	fwrite((char*)m,sizeof(mobile),1,f);
	return m;
}
void print_m(mobile *m){
	gotoxy(24,6);
	cprintf("RefId          - %d \n",m->refId);

	gotoxy(24,8);
	cprintf("Model          - %s \n",m->name);

	gotoxy(24,10);
	cprintf("Manufacturer   - %s \n",m->company);

	gotoxy(24,12);
	cprintf("IMEI           - %s \n",m->IMEI);

	gotoxy(24,14);
	cprintf("Price          - %f \n",m->price);
}

int high_x = 41;
int highlits[]={8,10,12,14};

void init (){
	mobile temp;
	f = fopen("data.dat","r+b");
	if(f == NULL){
		f = fopen("data.dat","w");  // Kind of like touch data.dat
		fclose(f);
		f = fopen("data.dat","r+b");
		printf("Created a new file");
	}
	if(f == NULL){
		clrscr();
		printf("Error Cant Load File");
		getch();
		exit(1);
	}
	while( fread((char*)&temp, sizeof(mobile), 1, f)){
		// Avoids max_overflows
		_refId = ((_refId > temp.refId )?_refId:temp.refId);
	}
	_refId++;
	clrscr();
	fseek(f,0,SEEK_SET); // Set to initial position.
}


void edit_m(mobile *m,int rec_in_strm){
	char ch = 0;
	float fr= 0.0;
	int high = 0, // Highlighted option
		counter=0; // File Counter
	while(1){
		prnt_scr();

		gotoxy(28,3);
		printf("--:: CURRENT DETAILS ::--");
		print_m(m);

		gotoxy(15,42);
		cprintf("Press Up / Down Arrow to Navigate Return To select");
		gotoxy(15,43);
		cprintf("Escape Twice. to Save & Exit");
		cprintf("rec_in_strm : %d",rec_in_strm);
		if(ch == 27){
			fseek(f,(sizeof(mobile)*rec_in_strm),SEEK_SET);
			fwrite((char*)m,sizeof(mobile),1,f);
			getch();
			return;
		}
		fflush(stdin);
		if(ch == 13){
			//if Ch == 13 then edit the option.
			gotoxy(high_x,highlits[high]);
			printf("                            ");
			gotoxy(high_x,highlits[high]);

			switch(high){
			case 0:
				gets(m->name);
				break;
			case 1:
				gets(m->company);
				break;
			case 2:
				gets(m->IMEI);
				break;
			case 3:
				fflush(stdin);
				scanf("%f",&fr);
				m->price = fr;//this is incredible lol
				fflush(stdin);
				break;
			default:
				printf("Error Encoutered! Possible Memory Leak");
				getch();
				exit(1);
			}

		}
		if(ch == 72 ){
			// Up Arrow
			high--;
		}else if(ch == 80){
			// Down Arrow
			high++;
		}
		high = ( (high < 0) ? (high+4) : high);
		high %= 4;
		textbackground( WHITE );
		textcolor( BLACK );
		gotoxy( high_x, highlits[high]);
		switch( high ){
	 		case 0:
			 cprintf("%s",m->name);
			 break;
	 		case 1:
			 cprintf("%s",m->company);
			 break;
	 		case 2:
			 cprintf("%s",m->IMEI);
			 break;
	 		case 3:
			 cprintf("%f",m->price);
			 break;
			default:
		 	 cprintf("Memory Error Escaping / Aborting");
			 exit(1);
		}

		textbackground(BLACK);
		textcolor(WHITE);
		ch = getch();

	}
}

typedef enum{
	MAIN,
	ADD,
	LIST,
	EDIT,
	SEARCH,
	DELETE,
	ABOUT
}MENUS;

int main_menu_y[]={8,10,12,14,16,18};

char main_menu_t[][20]={
	"Add",
	"List",
	"Search & Edit",
	"Delete a record",
	"About",
	"Exit"
};

void search(){
	mobile mob;
	int srch_id = -1,counter = 0;
	prnt_scr();
	fseek(f,0,SEEK_SET);

	gotoxy(22,20);
	cprintf("Please Enter the Ref. Id of the Handset");

	gotoxy(34,25);
	cprintf("Ref.Id : ");
	scanf("%d",&srch_id);
	fflush(stdin);

	while( fread((char*)&mob, sizeof(mob), 1, f) ){
		if( mob.refId == srch_id ){
			edit_m(&mob,counter);
			return;
		}
		counter++; // Adds the counter so that we know how many records to traverse before refeeding the file
	}
	gotoxy(22,30);
	textcolor(RED);
	cprintf("Sorry No Handset Exists for Ref.Id : %d",srch_id);
	textcolor(WHITE);
	getch();

}

void delete_mob(){

	FILE *f_temp;
	mobile mob;
	int srch_id = -1,
	flag    =  0;

	f_temp = fopen("temp.dat","wb");
	if(f_temp == NULL){
		textcolor(RED);
		clrscr();
		cprintf("ERROR OCCRURED ABORTING");
		exit(1);
	}
	prnt_scr();
	gotoxy(22,20);
	cprintf("Enter the Ref.Id of the record to delete");
	
	gotoxy(34,25);
	cprintf("Ref.Id : ");
	fflush(stdin);
	scanf("%d",&srch_id);
	
	fseek(f,0,SEEK_SET); // Beginning of file.

	while( fread((char*)&mob,sizeof(mob),1,f) ){
		if(mob.refId != srch_id){
			// Oops.
			fwrite((char*)&mob,sizeof(mob),1,f_temp);
		}else{
			// Skip the record.
			flag = 1;
		}
	}

	gotoxy(22,30);
	if(flag == 1){
		textcolor(GREEN);
		cprintf("Record Successfully Removed");
	}else{
		textcolor(RED);
		cprintf("No Record found for Ref.Id : %d nothing to delete");
	}
	textcolor(WHITE);

	fclose(f);
	fclose(f_temp);
	remove("data.dat");
	rename("temp.dat","data.dat");
	
	f = fopen("data.dat","r+b");

}

// PAGESIZE DEFINES HOW MANY RECORDS TO SHOW PER PAGE.
#define PAGESIZE 25

// Function to print all outputs

void show_list(){
	// Prints a nice table output of n outputs
	mobile temp;
	int printed=0;
	fseek(f,0,SEEK_SET);
	prnt_scr();

	gotoxy(2,1);
	cprintf("\n------------------------------------------------------------------------------");
	gotoxy(2,wherey());
	cprintf("\n|Ref.Id|     Name     |       Manufacturer      |  Price  |         IMEI     |");
	gotoxy(2,wherey());
	cprintf("\n------------------------------------------------------------------------------");

	while( fread((char*)&temp,sizeof(temp),1,f )){
		printed++;
		gotoxy(	2,wherey());
		cprintf("\n|%d|%*s|%*s|%.2f|%*s|",6,temp.refId,14,temp.name,25,temp.company,9,temp.price,18,temp.IMEI);
		gotoxy(2,wherey());
		cprintf("\n------------------------------------------------------------------------------");
		if( printed % PAGESIZE == 0 ){
			gotoxy(40,wherey()+2);
			cprintf("Press Any Key to Continue");
			getch();
			// Le New Page
			prnt_scr();
			gotoxy(2,1);
			cprintf("\n------------------------------------------------------------------------------");
			gotoxy(2,wherey());
			cprintf("\n|Ref.Id|     Name     |       Manufacturer      |  Price  |         IMEI     |");
			gotoxy(2,wherey());
			cprintf("\n------------------------------------------------------------------------------");

		}
	};
	getch();
}

// About function tells about.
void about(){
	prnt_scr();
	gotoxy(22,5);
	textcolor(YELLOW);
	cprintf(" This project was prepared by Abhishek "Darkyen" Hignikar during Esquare");
	gotoxy(22,6);
        cprintf(" training program under the guidance of ");
	textcolor(GREEN);
	cprintf("Meghna Raizada ");
	textcolor(YELLOW);
	cprintf("mam");
	
	gotoxy(22,8);
	cprintf("This project is distributed under MIT Licence");
	getch();
}
void main(){
	mobile * m;
	int menu,menu_h=0,i,key;
	
	init();
	highvideo(); // Super Dooper Eye Candy Looks!
	window(1,1,80,25);
	textmode(64);

	// Main Entry Point
	while(1){// Exit only on user action
		prnt_scr();
		gotoxy(30,3);
		cprintf("Main Menu");
		for(i = 0;i < 6 ;i++){
			gotoxy(30,main_menu_y[i]);
			if(i == menu_h){
				gotoxy(28,main_menu_y[i]);
				textcolor(YELLOW);
				cprintf("> %s",main_menu_t[i]);
				textcolor(WHITE);
			}else
				cprintf("%s",main_menu_t[i]);
		}
		
		gotoxy(15,42);
		cprintf("Press Up / Down Arrow to Navigate Return To select");
		
		gotoxy(15,43);
		cprintf("Escape to exit");
		
		key = getch();
		if(key == 27){
			exit(0);
		}else if(key == 13){
			switch(menu_h){
				case 0:
				 new_m();
				 break;
				case 1:
				 show_list();
				 break;
				case 2:
				 search();
				 break;
				case 3:
				 delete_mob();
				 getch();
				 break;
				case 4:
				 about();
				 break;
				case 5:
				 fclose(f);
				 exit(0);
				 break; // Overkill
				default:
				 clrscr();
				 textcolor(RED);
	  			 cprintf("Memory Leak Aborting");
				 exit(1);
			}
		}else if(key == 72){
			menu_h--;
		}else if(key == 80){
			menu_h++;
		}

		menu_h = ((menu_h<0)?(menu_h+6):menu_h);
		menu_h %= 6;
	}
}