+++
authors = "Johnny"
title = "An interesting source codes about Rance10"
date = "2026-04-24"
tags = [
    "c", 
    "operating system",
    "multi-thread programing"
]
+++


I found these codes when I was playing Rance10 and I think these could be a good studying profiles for c system programing  

  
**base.h**  

```   
#include<windows.h>
#include<process.h>
#include<stdlib.h>
#include<stdio.h>
#include<string.h>
#include <tlhelp32.h>
#include<dbghelp.h>
#define DefTimer 1000

//传入窗体类名，窗体存在返回TRUE，否则返回FALSE
BOOL  Find(char* ClassName){
    HWND hwnd = FindWindowA(ClassName, NULL);
    if (hwnd == 0)
		return FALSE;
	else
		return TRUE;
}

//线程树结构体
typedef struct{
    HANDLE TID;
    BOOL ALL_SET_UP;
    BOOL Quit;
    BOOL Pause;
    BOOL OutPut;
    BOOL Switch;
    char* Message;
    void (*Function)(void*);
    void* args;
    void* Next;
    int timer;
}ThreadTree;
typedef ThreadTree* BOX;
ThreadTree Ttree;
BOOL Quit=FALSE;
int Tcount=0;


//查询所在BOX
BOX GetBox(HANDLE TID){
    BOX Aim=&Ttree;
    for(int count=0;count<Tcount;count++){
        if(Aim->TID!=TID)
            Aim=(BOX)Aim->Next;
        else
            return Aim;
    }
    return NULL;
}


//标准线程
void StdThread(void* Box){
    BOX box=(BOX)Box;
    box->Message="线程上线\n";
    box->OutPut=TRUE;
    while(!box->Quit){
        while(!box->ALL_SET_UP);
        WaitForSingleObject(box->TID,box->timer);
        if(!box->Pause){
            box->Function(Box);
        }
    }
    box->Message="线程下线\n";
    box->OutPut=TRUE;
}


//消息线程
void MessageLoop(void* LPThreadTree){
    BOX Aim=(BOX)LPThreadTree;
    puts("[System]:消息循环上线");
    char Caption[10];
    //判断该次遍历是否有消息输出
    BOOL OutPut=FALSE;
    printf("[User]:");
    while(!Quit){
        for(int count=0;count<Tcount;count++){
            //任意消息队列存在输出就清除当前行
            if(Aim->OutPut){
                if(!OutPut){
                    OutPut=TRUE;
                    printf("\r");
                }
                sprintf(Caption,"%#x",Aim->TID);
                printf("[ThreadID#%s]:%s",Caption,Aim->Message);
                Aim->OutPut=FALSE;
            }
            Aim=(BOX)Aim->Next;
        }
        //恢复用户输入栏
        if(OutPut){
            printf("[User]:");
            OutPut=FALSE;
        }
        Aim=(BOX)LPThreadTree;
    }
    puts("[System]:消息循环下线");
}


//新建线程函数
HANDLE NewThread(void (*Function)(void*),void* args){
    if(!Tcount)
        _beginthread(MessageLoop,0,&Ttree);
    BOX Aim=&Ttree;
    for(int count=0;count<Tcount;count++)
        Aim=(BOX)Aim->Next;
    Aim->ALL_SET_UP=FALSE;
    Aim->Quit=FALSE;
    Aim->Pause=FALSE;
    Aim->OutPut=FALSE;
    Aim->Switch=FALSE;
    Aim->timer=DefTimer;
    Aim->args=args;
    //传入希望线程处理的函数
    Aim->Function=Function;
    Aim->Next=malloc(sizeof(ThreadTree));
    Aim->TID=(HANDLE)_beginthread(StdThread,0,Aim);
    Tcount++;
    Aim->ALL_SET_UP=TRUE;
    return Aim->TID;
}


//获取目标地址函数x32
DWORD GetAddress(char* ClassName,DWORD Ba_addr,int OffestCount,...){
    HWND hwnd = FindWindowA(ClassName, NULL);
    va_list args;//声明参数列表
    va_start(args,OffestCount);//以OffestCount为第一个参数地址
    if(hwnd!=NULL){
	        DWORD pid;
	        GetWindowThreadProcessId(hwnd, &pid);
	        HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
	        DWORD m_tempadd;
            DWORD m_offest;
            ReadProcessMemory(hProcess, (LPVOID)Ba_addr, &m_tempadd, 4, 0);
            for(int count=0;count<OffestCount-1;count++){
                m_offest=va_arg(args,DWORD);//从列表获取下一个参数
                ReadProcessMemory(hProcess, (LPVOID)(m_tempadd + m_offest), &m_tempadd, 4, 0);
            }
            return m_tempadd+va_arg(args,DWORD);
    }
    else return 0;
}


//获取模块地址的函数x32
DWORD GetModuleAddress(char* ClassName,char* ModuleName){
	HWND hwnd = FindWindowA(ClassName, NULL);
    if(hwnd!=NULL){
	    DWORD pid;
	    GetWindowThreadProcessId(hwnd, &pid);
	    HANDLE md_Handle;
	    md_Handle = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid);
	    if (!md_Handle)
		    return FALSE;
	    MODULEENTRY32 md;
	    md.dwSize = sizeof(md);
	    BOOL ex = Module32First(md_Handle, &md);
	    while (ex) {
		    if (!strcmp(md.szModule,ModuleName))
			    return (DWORD)md.modBaseAddr;
		    ex = Module32Next(md_Handle, &md);
	    }
    }
    else
	    return 0;
}


//用于GetModuleAddress64的结构体
typedef struct{
    DWORD64 addr;
    char* ModuleName;
}ModuleInfo;


//用于GetModuleAddress64的回调函数
BOOL CALLBACK MyEnumerateLoadedModulesProc64(
    PTSTR ModuleName,
    DWORD64 ModuleBase,
    ULONG ModuleSize,
    PVOID UserContext){
        ModuleInfo*AimModule=(ModuleInfo*)UserContext;
        if(strstr(ModuleName,AimModule->ModuleName)){
            AimModule->addr=ModuleBase;
            return 0;
            }
        return TRUE;
}


//获取模块地址的函数x64
DWORD64 GetModuleAddress64(char* ClassName,char* ModuleName){
    HWND hwnd = FindWindowA(ClassName, NULL);
    DWORD pid;
	GetWindowThreadProcessId(hwnd, &pid);
	HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
    HMODULE HDLL=LoadLibraryA("dbghelp.dll");
    BOOL (*ListMoudle)(HANDLE,PENUMLOADED_MODULES_CALLBACKW64,PVOID);
    ListMoudle=(BOOL (*)(HANDLE, PENUMLOADED_MODULES_CALLBACKW64, PVOID))GetProcAddress(HDLL,"EnumerateLoadedModules64");
    ModuleInfo AimModule;
    AimModule.ModuleName=ModuleName;
    ListMoudle(hProcess,(PENUMLOADED_MODULES_CALLBACKW64)MyEnumerateLoadedModulesProc64,&AimModule);
    CloseHandle(hProcess);
    return AimModule.addr;
}


//16进制打印地址函数
void AddrPrint(DWORD64 Aimer){
    char ku[16]={'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'};
    int zh[32],i=0,w,j;
    DWORD64 b;
    b=Aimer;
    w=16;
    do{
        zh[i]=Aimer%w;
        i++;
        Aimer/=w;
    }
    while(Aimer!=0);
    printf("0x");
    for(i--;i>=0;i--){
        j=zh[i];
        printf("%c",ku[j]);
    }
    printf("\n");
}


//跨进程申请内存空间函数
DWORD MallocEX(char* ClassName,int size){
    HWND hwnd = FindWindowA(ClassName, NULL);
    if(hwnd!=NULL){
        DWORD pid;
        GetWindowThreadProcessId(hwnd, &pid);
        HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
        DWORD addr=(DWORD)VirtualAllocEx(hProcess,NULL,size,MEM_COMMIT,PAGE_EXECUTE_READWRITE);
	    CloseHandle(hProcess);
	    if(addr==0)
		    return FALSE;
	    else return addr;
    }
    else
        return FALSE;
}


//跨进程释放内存地址函数
BOOL FreeEX(char* ClassName,DWORD addr){
    HWND hwnd = FindWindowA(ClassName, NULL);
    if(hwnd!=NULL){
        DWORD pid;
        GetWindowThreadProcessId(hwnd, &pid);
        HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
	    if(VirtualFreeEx(hProcess,(LPVOID)addr,0,MEM_RELEASE)==FALSE)
		    return FALSE;
	    else 
		    return TRUE;
    }
    else
        return FALSE;
}


//读取内存函数
BOOL ReadMemory(char* ClassName,DWORD m_addr,LPVOID Container,SIZE_T nSize){
    HWND hwnd = FindWindowA(ClassName, NULL);
    if(hwnd!=NULL){
	    DWORD pid;
	    GetWindowThreadProcessId(hwnd, &pid);
	    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
	    ReadProcessMemory(hProcess,(LPVOID)m_addr,Container,nSize,NULL);
	    CloseHandle(hProcess);
        return TRUE;
    }
    else
        return FALSE;
}


//内存写入函数
BOOL WriteMemory(char* ClassName,DWORD m_addr,LPVOID Value,SIZE_T nSize){
	HWND hwnd = FindWindowA(ClassName, NULL);
    if(hwnd!=NULL){
	    DWORD pid;
	    GetWindowThreadProcessId(hwnd, &pid);
	    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);
	    BOOL Result;
	    Result=WriteProcessMemory(hProcess,(LPVOID)m_addr, Value, nSize, &pid);
	    CloseHandle(hProcess);
        return Result;
    }
    else
	    return FALSE;
}

```  

 
**Rance10.c**  
  
```  
#include "base.h"
#define ClassName "AliceRunWindowClass"
#define AddrCount 13
#define DefHP 1000
#define DefBaseExp 40000000
typedef struct{
    DWORD* addr;
    char* Name;
}AddrContain;

AddrContain Addrs[AddrCount];
DWORD HPaddr;
DWORD BaseEXPaddr;
DWORD SlayAddr;
DWORD ComboAddr;
DWORD EXPaddr;
DWORD PointAddr;
DWORD TurnAddr;
DWORD VolumeAddr;
DWORD DeathRateAddr;
DWORD LisasAddr;
DWORD HermanAddr;
DWORD SethAddr;
DWORD LibertyAddr;


void AddrCheck(void* Box){
    BOX box=(BOX)Box;
    HPaddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB0;
    HPaddr=GetAddress(ClassName,HPaddr,5,0x8,0x438,0xc,0x10,0x10);
    Addrs[0].addr=&HPaddr;
    Addrs[0].Name="团队生命值";
    BaseEXPaddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB0;
    BaseEXPaddr=GetAddress(ClassName,BaseEXPaddr,5,0x8,0x520,0x8,0x10,0xc);
    Addrs[1].addr=&BaseEXPaddr;
    Addrs[1].Name="战斗基础经验值";
    SlayAddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB0;
    SlayAddr=GetAddress(ClassName,SlayAddr,5,0x8,0x478,0x4,0x10,0x20);
    Addrs[2].addr=&SlayAddr;
    Addrs[2].Name="必杀率";
    ComboAddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB0;
    ComboAddr=GetAddress(ClassName,ComboAddr,5,0x8,0x478,0x4,0x10,0x24);
    Addrs[3].addr=&ComboAddr;
    Addrs[3].Name="连击率";
    EXPaddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB0;
    EXPaddr=GetAddress(ClassName,EXPaddr,5,0x8,0x42c,0xc,0x10,0x1c);
    Addrs[4].addr=&EXPaddr;
    Addrs[4].Name="团队累积经验值";
    PointAddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB8;
    PointAddr=GetAddress(ClassName,PointAddr,5,0x150,0x4d4,0x4,0x10,0x8);
    Addrs[5].addr=&PointAddr;
    Addrs[5].Name="部队加成点数";
    TurnAddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB0;
    TurnAddr=GetAddress(ClassName,TurnAddr,5,0x8,0x420,0x0,0x10,0x8);
    Addrs[6].addr=&TurnAddr;
    Addrs[6].Name="回合数";
    VolumeAddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB0;
    VolumeAddr=GetAddress(ClassName,VolumeAddr,5,0x8,0x42c,0xc,0x10,0x0);
    Addrs[7].addr=&VolumeAddr;
    Addrs[7].Name="食卷/餐卷数";
    DeathRateAddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB4;
    DeathRateAddr=GetAddress(ClassName,DeathRateAddr,5,0x8,0x5b4,0xc,0x10,0x94);
    Addrs[8].addr=&DeathRateAddr;
    Addrs[8].Name="人类死亡率(单位0.1%)";
    LisasAddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB4;
    LisasAddr=GetAddress(ClassName,LisasAddr,5,0x8,0x5b4,0xc,0x10,0x64);
    Addrs[9].addr=&LisasAddr;
    Addrs[9].Name="利萨斯兵力";
    HermanAddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB4;
    HermanAddr=GetAddress(ClassName,HermanAddr,5,0x8,0x5b4,0xc,0x10,0x68);
    Addrs[10].addr=&HermanAddr;
    Addrs[10].Name="赫尔曼兵力";
    SethAddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB4;
    SethAddr=GetAddress(ClassName,SethAddr,5,0x8,0x5b4,0xc,0x10,0x6c);
    Addrs[11].addr=&SethAddr;
    Addrs[11].Name="塞斯兵力";
    LibertyAddr=GetModuleAddress(ClassName,"Rance10.exe")+0x003FCBB4;
    LibertyAddr=GetAddress(ClassName,LibertyAddr,5,0x8,0x5b4,0xc,0x10,0x70);
    Addrs[12].addr=&LibertyAddr;
    Addrs[12].Name="自由都市兵力";
}

void HealthManager(void* Box){
    BOX box=(BOX)Box;
    int HP;
    ReadMemory(ClassName,HPaddr,&HP,sizeof(int));
    if(HP<(int)box->args){
        box->Message="检测到生命值低于预设下限值,自动进行修正\n";
        box->OutPut=TRUE;
        WriteMemory(ClassName,HPaddr,&box->args,sizeof(int));
    }
}

void AttackMode(void* Box){
    int DefValue=100;
    BOX box=(BOX)Box;
    int Combo,Slay;
    ReadMemory(ClassName,ComboAddr,&Combo,sizeof(int));
    ReadMemory(ClassName,ComboAddr,&Slay,sizeof(int));
    if(Slay<100){
        WriteMemory(ClassName,SlayAddr,&DefValue,sizeof(int));
        box->Message="必杀率已修正\n";
        box->OutPut=TRUE;
        WaitForSingleObject(box->TID,500);
    }
    if(Combo<100){
        WriteMemory(ClassName,ComboAddr,&DefValue,sizeof(int));
        box->Message="连击率已修正\n";
        box->OutPut=TRUE;
    }
}

void FastLevelUP(void* Box){
    BOX box=(BOX)Box;
    int BaseExp;
    ReadMemory(ClassName,BaseEXPaddr,&BaseExp,sizeof(int));
    if(BaseExp<(int)box->args){
        box->Message="基础经验值已修正\n";
        box->OutPut=TRUE;
        WriteMemory(ClassName,BaseEXPaddr,&box->args,sizeof(int));
    }
}

int main(int argc,char*argv[]){
    system("chcp 65001");
    
    system("color 0a");
    system("title Rance10辅助控制台——编写由@A迷失的方向A");
    BOX Aim;
    HANDLE AddrCheck_Handle=NewThread(AddrCheck,NULL);
    HANDLE HP_Handle=NewThread(HealthManager,(void *)DefHP);
    HANDLE AttackMode_Handle=NewThread(AttackMode,NULL);
    HANDLE FastLevelUP_Handle=NewThread(FastLevelUP,(void *)DefBaseExp);
    Aim=GetBox(AttackMode_Handle);
    Aim->Pause=TRUE;
    Aim=GetBox(FastLevelUP_Handle);
    Aim->Pause=TRUE;
    BOOL End=FALSE;
    WaitForSingleObject(HP_Handle,500);
    Ttree.Message="生命值监测模块[HP]已上线\n";
    Ttree.OutPut=TRUE;
    WaitForSingleObject(AttackMode_Handle,500);
    Ttree.Message="Attack模块已上线,按默认设置待机\n";
    Ttree.OutPut=TRUE;
    WaitForSingleObject(FastLevelUP_Handle,500);
    Ttree.Message="LevelUP模块已上线,按默认设置待机\n";
    Ttree.OutPut=TRUE;
    WaitForSingleObject(HP_Handle,500);
    Ttree.Message="控制台已上线,输入./Help查看帮助\n";
    Ttree.OutPut=TRUE;
    while(!End){
        fflush(stdin);
        char input[10];
        char Cmd[3][10];
        char CmdCount=0;
        for(int count=0;count<10;count++){
            input[count]=toupper(getchar());
            if(input[count]==' '){
                input[count]='\n';
                sprintf(Cmd[CmdCount],"%s",input);
                for(int count=0;count<10;count++)
                    input[count]=0;
                count=-1;
                CmdCount++;
            }
            if(input[count]=='\n'){
                sprintf(Cmd[CmdCount],"%s",input);
                CmdCount++;
                break;
            }
        }
        switch(CmdCount){
            case 1:
                if(!strcmp("CLS\n",Cmd[0])){
                    system("cls");
                    printf("[User]:");
                    break;
                }
                if(!strcmp("HELP\n",Cmd[0])){
                    Ttree.Message="命令帮助页:\
                                                    \n        ./Cls 清除页面\
                                                    \n        ./Help 查看帮助页\
                                                    \n        ./List 查询所有值\
                                                    \n        ./Stop 停止运行\
                                                    \n        ./Switch <ModuleName> 切换模块运行状态\
                                                    \n              -可用的模块列表:\
                                                    \n                  ./HP 生命值检测模块\
                                                    \n                  ./Attack 攻击模式(自动补正连击率和必杀率)\
                                                    \n                  ./LevelUP 快速升级模块(自动修正基础经验值)\
                                                    \n        ./Set <ValueName> <Value> 设定一个对象的值\
                                                    \n              -可用的数值对象列表:\
                                                    \n                  ./BaseEXP 战斗基础经验值\
                                                    \n                  ./Combo 连击发生率\
                                                    \n                  ./dRate 人类死亡率(单位0.1%)\
                                                    \n                  ./DefHP 生命值监测的下限(默认为1000)\
                                                    \n                  ./EXP 团队累积经验值\
                                                    \n                  ./HP 团队生命值\
                                                    \n                  ./Herman 赫尔曼兵力值\
                                                    \n                  ./Lisas 利萨斯兵力值\
                                                    \n                  ./Seth 塞斯兵力值\
                                                    \n                  ./Liberty 自由都市兵力值\
                                                    \n                      -注:各兵力值单位为千\
                                                    \n                  ./Point 部队加成点数\
                                                    \n                  ./Slay 必杀发生率\
                                                    \n                  ./Turn 回合数\
                                                    \n                  ./Volume 食卷/餐卷数\
                                                    \n        ->于2020/08/16  由   @A迷失的方向A   编写                      \n";
                    Ttree.OutPut=TRUE;
                    break;
                }
                if(!strcmp("STOP\n",Cmd[0])){
                    End=TRUE;
                    break;
                }
                if(!strcmp("LIST\n",Cmd[0])){
                    printf("\r    -数值列表:\n");
                    for(int count=0;count<AddrCount;count++){
                        int Aim;
                        ReadMemory(ClassName,*(Addrs[count].addr),&Aim,sizeof(int));
                        printf("        -%s:%d\n",Addrs[count].Name,Aim);
                    }
                    Aim=GetBox(HP_Handle);
                    printf("        -生命检测模块的当前下限值:%d\n",(int)Aim->args);
                    printf("[User]:");
                    break;
                }
                Ttree.Message="未知的命令,输入Help查看帮助\n";
                Ttree.OutPut=TRUE;
                break;
            case 2:
                if(!strcmp("SWITCH\n",Cmd[0])){
                    if(!strcmp("HP\n",Cmd[1])){
                        Aim=GetBox(HP_Handle);
                        if(Aim->Pause){
                            Aim->Pause=FALSE;
                            Aim->Message="HP监测已恢复\n";
                        }
                        else{
                            Aim->Pause=TRUE;
                            Aim->Message="HP监测已关闭\n";
                        }
                        Aim->OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("ATTACK\n",Cmd[1])){
                        Aim=GetBox(AttackMode_Handle);
                        if(Aim->Pause){
                            Aim->Pause=FALSE;
                            Aim->Message="Attack模式已开启\n";
                        }
                        else{
                            Aim->Pause=TRUE;
                            Aim->Message="Attack模式已关闭\n";
                        }
                        Aim->OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("LEVELUP\n",Cmd[1])){
                        Aim=GetBox(FastLevelUP_Handle);
                        if(Aim->Pause){
                            Aim->Pause=FALSE;
                            Aim->Message="LevelUP模式已开启\n";
                        }
                        else{
                            Aim->Pause=TRUE;
                            Aim->Message="LevelUP模式已关闭\n";
                        }
                        Aim->OutPut=TRUE;
                        break;
                    }
                }
            Ttree.Message="未知的命令,输入Help查看帮助\n";
                Ttree.OutPut=TRUE;
                break;
            case 3:
                if(!strcmp("SET\n",Cmd[0])){
                    if(!strcmp("HP\n",Cmd[1])){
                        int HP;
                        sscanf(Cmd[2],"%d",&HP);
                        WriteMemory(ClassName,HPaddr,&HP,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("BASEEXP\n",Cmd[1])){
                        int BaseExp;
                        sscanf(Cmd[2],"%d",&BaseExp);
                        WriteMemory(ClassName,BaseEXPaddr,&BaseExp,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("SLAY\n",Cmd[1])){
                        int SlayPoint;
                        sscanf(Cmd[2],"%d",&SlayPoint);
                        WriteMemory(ClassName,SlayAddr,&SlayPoint,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("DEFHP\n",Cmd[1])){
                        int HPset;
                        sscanf(Cmd[2],"%d",&HPset);
                        Aim=GetBox(HP_Handle);
                        Aim->args=(void*)HPset;
                        Aim->Message="已修改生命值下限\n";
                        Aim->OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("COMBO\n",Cmd[1])){
                        int Combo;
                        sscanf(Cmd[2],"%d",&Combo);
                        WriteMemory(ClassName,ComboAddr,&Combo,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("EXP\n",Cmd[1])){
                        int EXP;
                        sscanf(Cmd[2],"%d",&EXP);
                        WriteMemory(ClassName,EXPaddr,&EXP,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("POINT\n",Cmd[1])){
                        int Point;
                        sscanf(Cmd[2],"%d",&Point);
                        WriteMemory(ClassName,PointAddr,&Point,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("TURN\n",Cmd[1])){
                        int Turn;
                        sscanf(Cmd[2],"%d",&Turn);
                        WriteMemory(ClassName,TurnAddr,&Turn,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("VOLUME\n",Cmd[1])){
                        int Volume;
                        sscanf(Cmd[2],"%d",&Volume);
                        WriteMemory(ClassName,VolumeAddr,&Volume,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("DRATE\n",Cmd[1])){
                        int DeathRate;
                        sscanf(Cmd[2],"%d",&DeathRate);
                        WriteMemory(ClassName,DeathRateAddr,&DeathRate,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("LISAS\n",Cmd[1])){
                        int Lisas;
                        sscanf(Cmd[2],"%d",&Lisas);
                        WriteMemory(ClassName,LisasAddr,&Lisas,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("HERMAN\n",Cmd[1])){
                        int Herman;
                        sscanf(Cmd[2],"%d",&Herman);
                        WriteMemory(ClassName,HermanAddr,&Herman,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("SETH\n",Cmd[1])){
                        int Seth;
                        sscanf(Cmd[2],"%d",&Seth);
                        WriteMemory(ClassName,SethAddr,&Seth,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                    if(!strcmp("LIBERTY\n",Cmd[1])){
                        int Liberty;
                        sscanf(Cmd[2],"%d",&Liberty);
                        WriteMemory(ClassName,LibertyAddr,&Liberty,sizeof(int));
                        Ttree.Message="已修改目标值\n";
                        Ttree.OutPut=TRUE;
                        break;
                    }
                }
            default :
                Ttree.Message="未知的命令,输入Help查看帮助\n";
                Ttree.OutPut=TRUE;
        }
        for(int count=0;count<10;count++)
            input[count]=0;
    }
    Ttree.Message="主线程下线\n";
    Ttree.OutPut=TRUE;
    WaitForSingleObject(HP_Handle,500);
    system("pause");
    return 0;
}  

```  

  
Currently I do not know much about the meaning about codes, but it is possible to learn about the secrets behind them