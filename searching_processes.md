#Поиск процессов в дампе памяти

Каждый ОС-процесс в Windows представлен структурой *\_EPROCESS*. Данная структура содержит атрибуты, 
описывающие процесс, а также другие связанные с ним структуры данных (например, для описания потоков процессов используется структура *\_ETHREAD*). Данные структуры *\_EPROCESS* хранятся в адресном пространстве ядра, за исключением блока переменных окружения (process environment block (PEB)). Первым элементом структуры процесса является блок управления процессом, представленный структурой *\_KPROCESS* и хранящейся также в пространстве ядра. Структуры *\_KPROCESS* и *\_KTHREAD* начинаются с субструктуры, известной как *\_DISPATCHER\_HEADER*. В данной подструктуре содержатся атрибуты _TYPE_ и _SIZE_, определяющие тип и размер объекта соответственно, которые принимают одинаковые значения для всех процессов конкретной операционной системы [1].
Для Windows 7 значение поля _TYPE_ равно **0x03**, значение поля _SIZE_ - **0x26**. Найдя данные значения в образе памяти – в адресном пространстве ядра, можно найти структуру *\_EPROCESS*.

Одним из недостатков данного подхода является тот факт, что с помощью данного метода можно найти только структуры процессов и потоков. Найдя структуру процесса, по значению поля *DirectoryTableBase*, хранящегося в структуре *\_KPROCESS*, можно получить физический адрес таблицы преобразования четвертого уровня PML4, а по значениям полей *ThreadListHead.Flink* и *ThreadListHead.Blink* виртуальные адреса предыдущего и последующего процессов соответственно. Данные поля должны содержать значения, соответствующие адресному процессу ядра [2].
```
typedef struct _EPROCESS {
+0x000 Pcb _KPROCESS {
+0x000     Header DISPATCHER_HEADER;
+0x010     ProfileListHead LIST_ENTRY;
+0x018     ULONG DirectoryTableBase;
...
       }
...
+0x308 ThreadListHead LIST_ENTRY {
+0x000 Flink PLIST_ENTRY; 
+0x004 Blink PLIST_ENTRY;
...
}
```
##Литература:



[1] Michael Hale Ligh, Andrew Case, Jamie Levy, AAron Walters, The Art of Memory Forensics: Detecting Malware and Threats in Windows,
 Linux, and Mac Memory, P.146-147



[2] Andreas Schuster, Searching for processes and threads in Microsoft Windows memory dumps, Digital Investigation, 2006, 
Vol. 3, P. 10-16.


