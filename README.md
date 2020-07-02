仓库管理系统
====
需求分析
------

仓库管理系统的功能如下：
1. 在仓库进货时，如果仓库中没有此商品，则为仓库增添新的商品项目
2. 在仓库进货时，如果仓库中已有此商品，则增加此商品的库存量
3. 在仓库出货时，减少对应商品的库存量
4. 在仓库出货时，如果这是货物是此商品的最后一批货（库存量为 0），则删除仓库中 此商品项目 
5. 查询功能：可以随时查看当前仓库的库存，包括商品名和剩余量 

实现思路
-----
功能结构图： 
 
本仓库管理系统主要分为出货、进货、查询三大模块，分别对出货、进货和查询的操作进行管理。
进货模块中又细分为增加库存和新增商品子功能，当进货时，若此商品在仓库中没有库存， 则在仓库库存条目中新增此商品项目，若已有此类商品，则根据进货量增加对应的库存量。
出货模块中又细分为减少库存和删除商品子功能，当出货时，减少对应商品的数目，注意到 当库存不足时，出货失败，且若出货成功并且库存为 0 时，删除仓库目录中此商品项目。
查询模块，分为显示库存和查找某一商品是否存在库存及其库存量 。

数据设计
---------
 typedef struct goods {   char name[100]; //记录货物名  
                           int count;  //记录货物数量
                        }Goods; 
函数设计
--------
 进货：
 -----
 //进货，对应进货模块，表示当前进货一批数量为 count 的 name 商品 
 bool add_goods(char name[], int count);//进货函数声名
 bool add_goods(char name[], int count){  int index=1; 
 if(increase_count(name, count)==0)//判断仓库中是否存在该货物，若存在，则增加 货物数量  
  add_to_list(name,count);//若不存在，则增加库存的货物种类  
  return index; }
  1、//更新库存信息，对应增加库存子功能，对 name 商品新增 count 数量 
  bool increase_count (char name[]，int count);//声名函数名称 
  bool increase_count(char name[], int count){  
  int goods_count=0;//定义货物数量  
  FILE * pgoods=fopen("goods.dat","r");//打开库存文件  
  FILE * pgoods1=fopen("temp.dat", "w"); //创建临时文件 
  if(pgoods==NULL||pgoods1==NULL)  printf("系统打开失败");//确保文件顺利打开  
  char * p1=name;  int index=0;  rewind(pgoods);//确保文件指针指向文件开头  
  rewind(pgoods1);  fread(&goods_count, sizeof(int), 1, pgoods);//读出文件中记录的货物种类数量  
  fwrite(&goods_count,sizeof(int), 1, pgoods1);//将货物种类数量写入临时文件中 
  Goods * p=(Goods *)malloc(sizeof(Goods)); 
  int i=0;  
  while(i<goods_count){  
  fread(p, sizeof(Goods), 1, pgoods); 
   if(stricmp(p1,p->name)==0){   
   p->count+=count;    
   index=1;  
   }   
   fwrite(p, sizeof(Goods), 1, pgoods1);   
   i++;
   }//将原库存文件中的数据复制到临时文件中，并更新进货的货物数量  
   fclose(pgoods);  fclose(pgoods1);//关闭两个文件  
   remove("goods.dat");  
   rename("temp.dat","goods.dat");//删除原库存文件，并将临时文件改名使其成为库存文 件，从而达到更新库存的目的  
   return index; 
   }   
   2、//更新库存列表，对应新增商品子功能，新增 name 商品且初始数量为 count  
   bool add_to_list(char name[], int count); //声明函数名称  
   bool add_to_list(char name[],int count){  
   int goods_count=0;  
   FILE * pgoods=fopen("goods.dat","r");  
   FILE * pgoods1=fopen("temp.dat", "w");//打开库存文件，并创建临时文件用于更新库存  
   if(pgoods==NULL)  
   printf("系统打开失败");//确保文件顺利打开  
   rewind(pgoods);  
   rewind(pgoods1);//确保文件指针指向文件开头  
   fread(&goods_count, sizeof(int ), 1, pgoods);  
   goods_count+=1;  
   fwrite(&goods_count, sizeof(int), 1, pgoods1);//读出原文件中的货物种类数量，并更新后 写入临时文件  
   int index=1;  
   Goods *p=(Goods *)malloc(sizeof(Goods));  
   for(int i=0; i<goods_count-1; i++){
   fread(p, sizeof(Goods), 1, pgoods);   
   fwrite(p, sizeof(Goods), 1, pgoods1);  
   } 
   fclose(pgoods);//将原文件中的库存数据复制到临时文件中  
   remove("goods.dat");  
   strcpy(p->name, name); 
   p->count=count; 
   fwrite(p, sizeof(Goods), 1, pgoods1);//将新增货物的数据写入临时文件中  
   fclose(pgoods1);  
   rename("temp.dat", "goods.dat");//将临时文件改名后成为库存数据文件，从而更新库存数据  
   return index; 
   }   
   
   出货：
   -----
   //出货，对应出货模块，表示当前出货一批数量为 count 的 name 商品 
   bool delete_goods(char name[], int count);//声名文件名 
   bool delete_goods(char name[], int count){  
   int index=1; 
    if(decrease_count(name, count)==0)//判断库存是否足够供给  
    delete_from_list(name);//若库存为 0 将货物数据删除  
    return index; 
    } 
    1、//更新库存信息，对应减少库存子功能，对 name 商品减少 count 数量  
    bool decrease_count(char name[], int count);
    bool decrease_count(char name[], int count){  
    int goods_count=0;  
    FILE * pgoods=fopen("goods.dat","r"); 
    FILE * pgoods1=fopen("temp.dat", "w"); //打开原文件，并创建临时文件用于更新数据  
    if(pgoods==NULL||pgoods1==NULL) 
    printf("系统打开失败");//确保文件能顺利打开  
    char * p1=name;  
    int index=0;  
    rewind(pgoods);  
    rewind(pgoods1); 
    fread(&goods_count, sizeof(int), 1, pgoods);  
    fwrite(&goods_count,sizeof(int), 1, pgoods1);//读出原文件中货物种类数量，并写入临时件中  
    Goods * p=(Goods *)malloc(sizeof(Goods));  
    int i=0;  
    while(i<goods_count){  
    fread(p, sizeof(Goods), 1, pgoods); 
   if(stricmp(p1,p->name)==0){    
   p->count-=count;    
   if(p->count>0)    
   index=1;    
   else if(p->count<0){     
   printf("库存不足！");     
   printf("\n");     
   printf("出货失败!");     
   printf("\n");     
   index=1;   
      } 
   }   
   fwrite(p, sizeof(Goods), 1, pgoods1);   
   i++; 
   }//将原文件中的库存数据复制到临时文件中，并判断库存是否达到出货要求，同时更新出货后的数据  
   fclose(pgoods);  
   fclose(pgoods1);  
   remove("goods.dat");  
   rename("temp.dat","goods.dat");//将临时文件重新命名，使其成为库存文件，达到更新数据的目的 
   return index;
   } 
   2、//更新库存列表，对应删除商品子功能，删除商品列表中 name 商品  
   bool delete_from_list(char name[]); 
   bool delete_from_list (char name[]){  
   int goods_count=0;  
   FILE * pgoods=fopen("goods.dat","r");  
   FILE * pgoods1=fopen("temp.dat", "w");//打开就库存文件并创建临时文件用于更新数据  
   if(pgoods==NULL||pgoods1==NULL)  
   printf("系统打开失败");//确保文件打开成功  
   int index=1;  
   char *p1=name;  
   Goods *p=(Goods *)malloc(sizeof(Goods));  
   rewind(pgoods);  
   rewind(pgoods1);  
   fread(&goods_count, sizeof(int), 1,pgoods);  
   goods_count-=1;  
   fwrite(&goods_count,sizeof(int), 1,pgoods1);//读出原文件中的货物种类数据并进行更新后写入临时文件  
   int i=0; 
   for(i=0;i<goods_count+1;i++){  
   fread(p, sizeof(Goods), 1, pgoods);   
   if(stricmp(p1, p->name)!=0){    
   fwrite(p, sizeof(Goods), 1, pgoods1); 
   } 
   }//将原文件中的库存数据复制到临时文件中，需要删除的货物的库存数据不写入临时文件从而更新数据  
   fclose(pgoods);  
   fclose(pgoods1);  
   remove("goods.dat");  
   rename("temp.dat","goods.dat");//临时文件改名后成为库存数据文件  
   return index; 
   }
 
 查询：
 -----
 1、///显示当前库存列表，包括商品名及其库存量  
 void show_goods();  
 void show_goods(){  
 int goods_count=0;  
 FILE * pgoods=fopen("goods.dat","r");  
 if(pgoods==NULL)  
 printf("系统打开失败");//确保库存文件能打开  
 rewind(pgoods);  
 fread(&goods_count, sizeof(int ), 1, pgoods); 
 Goods *p=(Goods *)malloc(sizeof(Goods));  
 for(int i=0;i<goods_count;i++){   
 fread(p, sizeof(Goods), 1, pgoods);   
 printf("%s: %d\n", p->name, p->count);  
 }//遍历库存数据，将货物的数据输出  
 fclose(pgoods); 
 } 
 2、//查看仓库中的 name 商品  
 struct Goods find_goods(char name[]); 
 Goods find_goods(char name[]){  
 int goods_count=0;  
 FILE * pgoods=fopen("goods.dat","r");  
 if(pgoods==NULL)  
 printf("系统打开失败");//确保库存文件能打开  
 char *p1=name;  
 rewind(pgoods);  
 fread(&goods_count, sizeof(int), 1, pgoods);  
 Goods *p=(Goods*)malloc(sizeof(Goods));  
 for(int i=0;i<goods_count;i++){   
 fread(p,sizeof(Goods), 1, pgoods);   
 if(stricmp(p1,p->name)==0){    
 break;   
 }  
 }//遍历库存文件的数据，找到需要查询的货物数据  
 fclose(pgoods);  
 return *p;//返回货物数据 
 } 
 
使用说明 
--------
1、想要进货，输入数字 1，并输入货物名称和进货数量 
2、想要出货，输入数字 2，并输入货物名称和进货数量，若库存不足，出货失败，若库存 为零，则删除库存中该货物 
3、想要查询库存情况，输入数字 3；可查询其中一件货物的库存，请输入 1 并输入货物的 名称；想查询所有库存情况，输入数字 2 4、想退出仓库管理系统，输入数字 0 

输入与输出
-------
【输出】 
成功打开系统！ 
如果你想要进货，请输入 1， 出货，请输入 2， 查询，请输入 3 
如果想要关闭系统，请输入 0 

【输入】 
1 

【输出】 
请输入需要进货的货物的名称及其数量 

【输入】 
Apple 200 

【输出】 
如果你想要进货，请输入 1， 出货，请输入 2， 查询，请输入 3 如果想要关闭系统，请输入 0 

【输入】 
3 

【输出】
若想要查看仓库中的一件商品，请按 1 并输入其名称，若想要查看当前库存列表，请按 2 

【输入】 
2 

【输出】
pen: 400 
banana: 100 
apple: 400 
pear: 200 

【输出】 
如果你想要进货，请输入 1， 出货，请输入 2， 查询，请输入 3 如果想要关闭系统，请输入 0 

【输入】 
0 

【输出】 
系统关闭！ 
再见！ 
