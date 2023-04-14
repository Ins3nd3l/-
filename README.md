# 
разработать приложение, которое предоставляет пользователю возможность управлять памятью в Windows, получать системную информацию о состоянии физической и виртуальной памяти, а также управлять выделением и освобождением виртуальной памяти. на языке c++
#pragma hdrstop
#pragma argsused
 
#include <tchar.h>
  typedef char _TCHAR;
  #define _tmain main
 
 
#include <stdio.h>
 
//состояние страниц
switch(mem_info2.State)
{
case MEM_COMMIT:
i+=sprintf(buf+i," COMMIT ");
break;
case MEM_FREE:
i+=sprintf(buf+i," FREE ");
break;
case MEM_RESERVE:
i+=sprintf(buf+i," RESERVE");
break;
default:
i+=sprintf(buf+i," UNKNOWN");
break;
}
 
VirtualQuery(pMemory, &memInfo,
sizeof(MEMORY_BASIC_INFORMATION));
switch(memInfo.Protect)
{
case PAGE_EXECUTE:
SendDlgItemMessage(hDlg, IDC_RADIO1, BM_SETCHECK,
BST_CHECKED, NULL);
break;
case PAGE_EXECUTE_READ:
SendDlgItemMessage(hDlg, IDC_RADIO2, BM_SETCHECK,
BST_CHECKED, NULL);
break;
case PAGE_EXECUTE_READWRITE:
SendDlgItemMessage(hDlg, IDC_RADIO3, BM_SETCHECK,
BST_CHECKED, NULL);
break;
case PAGE_EXECUTE_WRITECOPY:
SendDlgItemMessage(hDlg, IDC_RADIO4, BM_SETCHECK,
BST_CHECKED, NULL);
break;
case PAGE_NOACCESS:
SendDlgItemMessage(hDlg, IDC_RADIO5, BM_SETCHECK,
BST_CHECKED, NULL);
break;
case PAGE_READONLY:
SendDlgItemMessage(hDlg, IDC_RADIO6, BM_SETCHECK,
BST_CHECKED, NULL);
break;
case PAGE_READWRITE:
23
SendDlgItemMessage(hDlg, IDC_RADIO7, BM_SETCHECK,
BST_CHECKED, NULL);
break;
case PAGE_WRITECOPY:
SendDlgItemMessage(hDlg, IDC_RADIO8, BM_SETCHECK,
BST_CHECKED, NULL);
break;
}
 
   INT_PTR CALLBACK MemoryInfo(HWND hDlg, UINT message,
WPARAM wParam, LPARAM lParam)
{
UNREFERENCED_PARAMETER(lParam);
SYSTEM_INFO sys_info;//структура, содержащая
//системную информацию (размер страницы, гранулярность
// выделения памяти и др.0
MEMORYSTATUS mem_status = { sizeof(mem_status) };
char buff[256];
string str1="Размер страницы памяти:";
string str2="Байт свободно в страничном файле:";
switch (message)
{
case WM_INITDIALOG:
SetTimer(hDlg,TIMER,1*500,NULL);
SendMessage(hDlg, WM_TIMER, NULL, NULL);
GetSystemInfo(&sys_info);
str1=str1+itoa(sys_info.dwPageSize,buff,10)+"\r\n";
sprintf(buff,"Максимальный адрес доступной памяти:
%p",sys_info.lpMaximumApplicationAddress);
str1=str1+buff+"\r\n";
     sprintf(buff,"Минимальный адрес доступной памяти:
%p",sys_info.lpMinimumApplicationAddress);
str1=str1+buff+"\r\n";
sprintf(buff, TEXT("0x%016I64X"), ( int64)
sys_info.dwActiveProcessorMask);
str1=str1+"Число процессоров:
"+itoa(sys_info.dwNumberOfProcessors,buff,10)+"\r\n";
str1=str1+"Гранулярность резервирования:
"+buff+"\r\n";
SetDlgItemText(hDlg,IDC_SYSTEM_INFO,str1.c_str());
return (INT_PTR)TRUE;
case WM_TIMER:
GlobalMemoryStatus(&mem_status);//позволяет отслеживать текущее состояние памяти
sprintf(buff,"%I64d",( int64)
mem_status.dwAvailPageFile);
str2=str2+buff+"\r\n";
sprintf(buff,"%d", mem_status.dwTotalPhys);
str2=str2+"Объем физической памяти в байтах:
"+buff+"\r\n";
sprintf(buff,"%I64d",( int64)
mem_status.dwAvailPhys);
str2=str2+"Число байтов свободной физической памяти:
"+buff+"\r\n";
sprintf(buff,"%I64d",( int64)
mem_status.dwTotalVirtual);
str2=str2+"Объем виртуальной памяти в байтах):
"+buff+"\r\n";
sprintf(buff,"%I64d",( int64)
mem_status.dwAvailVirtual);
str2=str2+"Число байтов свободной виртуальной памяти:
"+buff+"\r\n";
sprintf(buff,"%I64d",( int64)
mem_status.dwTotalPageFile);
str2=str2+"Макс. кол-во байт в стран. файле на hdd:
"+buff+"\r\n";
sprintf(buff,"%d",mem_status.dwMemoryLoad);
str2=str2+"Насколько занята подсистема управления памятью: "+buff+"\r\n";
SetDlgItemText(hDlg,IDC_MEMORY_STATUS,str2.c_str());
return TRUE;
}
return (INT_PTR)FALSE;
 
case IDC_ALLOC:
pMemory1= (char*)GetDlgItemInt(hDlg,IDC_ADRESS,
&bResult, FALSE);
if ( pMemory == NULL )
{
size = GetDlgItemInt(hDlg,IDC_SIZE, &bResult, FALSE);
if ( bResult && size>0)
pMemory = (char*)VirtualAlloc(
//возвращает виртуальный адрес региона
pMemory1,
//система должна зарезервировать регион там, где,
//по ее мнению, будет лучше
size , //размер резервируемого региона в байтах
MEM_RESERVE |
//зарезервировать регион адресного пространства
MEM_COMMIT | //Для передачи физической памяти
MEM_TOP_DOWN,
//зарезервировать регион по самым старшим адресам
PAGE_READWRITE);//указывает атрибут защиты
if( pMemory == NULL )
MessageBox( hDlg, "Нельзя выделить память", "Ошибка",
MB_ICONERROR);
else
{
SendDlgItemMessage(hDlg, IDC_LIST, LB_ADDSTRING,
NULL, (LPARAM)"Память выделена");
VirtualQuery(//заполняет структуру MEMORY_BASIC_INFORMATION
//информацией о диапазоне смежных страниц, имеющих
//одинаковые состояние, атрибуты защиты и тип.
pMemory,//виртуальный адрес региона
&memInfo,//адрес структуры MEMORY_BASIC_INFORMATION
sizeof(MEMORY_BASIC_INFORMATION));//размер структуры
// MEMORY_BASIC_INFORMATION
 
// освобождение памяти
case IDC_FREE_MEMORY:
if(pMemory != NULL)
{
VirtualFree (pMemory,//базовый адрес региона
0, // вернет системе весь диапазон выделенных страниц
MEM_RELEASE);//возврат системе всей физической памяти,
//отображенной на регион, и освобождение самого региона
int count = SendDlgItemMessage(hDlg, IDC_LIST, LB_GETCOUNT,
NULL, NULL);
for (int i = 0; i< count; i++ )
SendDlgItemMessage(hDlg, IDC_LIST, LB_DELETESTRING, 0, NULL);
SendDlgItemMessage(hDlg, IDC_LIST, LB_ADDSTRING, NULL,
(LPARAM)"Память не выделена");
pMemory = NULL;
}
else MessageBox(hDlg,"Блок памяти не выделен","Ошибка",
MB_ICONEXCLAMATION);
return TRUE;
 
VirtualQuery(pMemory,&memInfo,sizeof(MEMORY_BASIC_INFORMATION));
 
// Изменение атрибута защиты
case IDC_ATTRIBYTE:
if (pMemory == NULL)
{
MessageBox(hDlg, "Память не выделена", "Ошибка",
MB_ICONEXCLAMATION);
}
if( SendDlgItemMessage(hDlg, IDC_RADIO1, BM_GETCHECK, NULL,
NULL) == BST_CHECKED)
dwProtect = PAGE_EXECUTE;
if( SendDlgItemMessage(hDlg, IDC_RADIO2, BM_GETCHECK, NULL,
NULL) == BST_CHECKED)
dwProtect = PAGE_EXECUTE_READ;
if( SendDlgItemMessage(hDlg, IDC_RADIO3, BM_GETCHECK, NULL,
NULL) == BST_CHECKED)
dwProtect = PAGE_EXECUTE_READWRITE;
if( SendDlgItemMessage(hDlg, IDC_RADIO4, BM_GETCHECK, NULL,
NULL) == BST_CHECKED)
dwProtect = PAGE_EXECUTE_WRITECOPY;
if( SendDlgItemMessage(hDlg, IDC_RADIO5, BM_GETCHECK, NULL,
NULL) == BST_CHECKED)
dwProtect = PAGE_NOACCESS;
  if( SendDlgItemMessage(hDlg, IDC_RADIO6, BM_GETCHECK, NULL,
NULL) == BST_CHECKED)
dwProtect = PAGE_READONLY;
if( SendDlgItemMessage(hDlg, IDC_RADIO7, BM_GETCHECK, NULL,
NULL) == BST_CHECKED)
dwProtect = PAGE_READWRITE;
if( SendDlgItemMessage(hDlg, IDC_RADIO8, BM_GETCHECK, NULL,
NULL) == BST_CHECKED)
dwProtect = PAGE_WRITECOPY;
if( !VirtualProtect( pMemory,// базовый адрес памяти
size,//число байтов, для которых изменяете атрибут защиты
dwProtect, //новый атрибут защиты
&dwOldProtect))//адрес переменной типа DWORD,
//в которую VirtualProtect заносит
//старое значение атрибута защиты
MessageBox( hDlg, "Не удалось изменить атрибуты защиты",
"Ошибка", MB_OK);
break;
 
// блокировка памяти
case IDC_LOCK_MEMORY:
if(pMemory != NULL)
{
if ( bLock == false)
{
VirtualQuery(pMemory,&memInfo,sizeof(MEMORY_BASIC_INF
ORMATION));
if (memInfo.Protect != PAGE_NOACCESS)
{
if (VirtualLock( memInfo.BaseAddress, size))
// Указатель на базовый адрес региона и размер региона
{
SendDlgItemMessage(hDlg, IDC_LIST, LB_ADDSTRING,
NULL, (LPARAM)"Блокировка памяти: ");
sprintf(buf, _TEXT("%d%s%Xh"), size, " байт начиная
с адреса : ", memInfo.BaseAddress );
SendDlgItemMessage(hDlg, IDC_LIST, LB_ADDSTRING,
NULL, (LPARAM)buf);
bLock = true;
}
else MessageBox(hDlg, "Слишком большой размер блока
памяти", "Ошибка", MB_ICONEXCLAMATION);
}
else MessageBox(hDlg, "Нет доступа к памяти", "Ошибка",
MB_ICONEXCLAMATION);
}
else MessageBox(hDlg, "Память уже заблокирована", "Ошибка",
MB_ICONEXCLAMATION);
}
else MessageBox(hDlg, "Память не выделена", "Ошибка",
MB_ICONEXCLAMATION);
return TRUE;
 
// разблокировка памяти
case IDC_UNLOCK_MEMORY:
if(pMemory != NULL)
{
if ( bLock )
{
VirtualQuery(pMemory,&memInfo,sizeof(MEMORY_BASIC_INFORMATION));
if (memInfo.Protect != PAGE_NOACCESS)
{
if (VirtualUnlock( memInfo.BaseAddress, size))
// Указатель на базовый адрес регионаи размер региона
{
SendDlgItemMessage(hDlg, IDC_LIST, LB_ADDSTRING,
NULL, (LPARAM)"Разблокировка памяти: ");
sprintf(buf, _TEXT("%d%s%Xh"), size, " байт начиная с
адреса : ", memInfo.BaseAddress );
SendDlgItemMessage(hDlg, IDC_LIST, LB_ADDSTRING,
NULL, (LPARAM)buf);
bLock = false;
}
}
else MessageBox(hDlg, "Нет доступа к памяти", "Ошибка",
MB_ICONEXCLAMATION);
}
else MessageBox(hDlg, "Память не заблокирована", "Ошибка",
MB_ICONEXCLAMATION);
}
else MessageBox(hDlg, "Память не выделена", "Ошибка",
MB_ICONEXCLAMATION);
return TRUE;
 
case IDC_CREATE_HEAP:
if (hHeap == NULL)
{
hsize = GetDlgItemInt(hDlg, IDC_EDIT_HEAP, &bResult,
FALSE);
if ( bResult && hsize>0 )
{
hHeap = HeapCreate( HEAP_NO_SERIALIZE, hsize, hsize);
SendDlgItemMessage(hDlg, IDC_LIST2, LB_ADDSTRING,
NULL, (LPARAM)"Создана новая куча ");
lpHeap = NULL;
}
else MessageBox(hDlg, "Неверно задан размер", "Ошибка", MB_ICONEXCLAMATION);
}
else MessageBox(hDlg, "Куча уже создана", "Ошибка",
MB_ICONEXCLAMATION);
return true;
 
 case IDC_DESTROY_HEAP:
if (hHeap != NULL)
{
HeapDestroy(hHeap);
int count = SendDlgItemMessage(hDlg, IDC_LIST,
LB_GETCOUNT, NULL, NULL);
for (int i = 0; i< count; i++ )
SendDlgItemMessage(hDlg, IDC_LIST2, LB_DELETESTRING,
0, NULL);
SendDlgItemMessage(hDlg, IDC_LIST2, LB_ADDSTRING,
NULL, (LPARAM)"Новая куча не создана");
hHeap = NULL;
}
else MessageBox(hDlg, "Куча еще не создана", "Ошибка", MB_ICONEXCLAMATION);
return true;
