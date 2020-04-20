# Sonata IC Verification Tool
This verification tool is for Sonata IC to modify register information.

## Tool Released Note
- V1.01 – First released </br>
- V1.02 – fix ddsm_ctrl2 register range </br>
- V1.03 – add preview、delay command、refresh button and fix window layer issue</br>
- V1.04 – add RG_BISTPLL_METER_CTRL Register、add value check mechanism </br>
- V1.05 – add RG_BISTPLL_METER_CTRL Register in export table、add value check mechanism (avoid syntax type error) </br>
- V1.06 – fix issue of import file (syntax error) </br>
- V1.07 – add BBPLL control register (M3) </br>
- <font color = F16773> V1.08 – fix RG_CKO_CTRL bit range and add register binary mode </font></br>

## System Requirments
 - libusb-win32 (v 1.2.6.0) driver</br>
   - This is FTDI driver. User have to use Zadig tool to install. 
     - driver安裝步驟請參考 "FT4232H_Driver安裝步驟(檔案放在public分享區，路徑可以參考Reference內容)
 - Python 3.5 or above
 - libusb, current tested with 1.0.23
 - PyFrdi is beeing tested with PyUSB 1.0.2
 - Windows 7 or above (32bit/64bit)

## Package Installation
```python
pip install pyftdi
```

 - 其他package請依照K-script所需使用的進行安裝(ex. StrEnum, pandas, numpy, bitstring, pyVISA)
 - pyftdi version: 0.42.2
 - pyusb version: 1.0.2

## Using pyinstaller bundles a Python applicaion
```python
pyinstaller .\xxx.spec
```
 - xxx依照個人設定的spec名稱
 - 打包完畢後，要將以下資料夾複製到dist內
   - Script_Sonata
   - Sonata (請移除python檔案)
   - image
 - 如果要重新create spec file，要注意以下項目
   - a = Analysis中binaries的中括號內填入('C:\\Windows\\System32\\libusb0.dll', '.'),
   - a = Analysis中hiddenimports的中括號內填入'usb'
   - 填好後再重新執行打包指令pyinstaller xxx.spec (xxx依照個人設定的spec名稱)

## Notices
 - USB裝置第一次連線時，必須先安裝FT4232H的driver
   - 同一裝置driver只需安裝一次
   - 安裝FT4232H driver之後，舊版Tool就會無法使用，如需使用舊版Tool，則需移除FT4232H driver (無法互通的原因是因為舊版Tool使用的是windows官方提供的driver，但因為搭配python套件-PyFtdi的driver無官方提供版，必須另外安裝，會導致與原先的有衝突，故有此現象會發生)
 - 若必須使用Sonata FPGA搭配debug，請留意以下:
   - 注意小板子的SPI接線位置 (不一定每塊相同)</br>
     <img src="https://i.imgur.com/gkd4eKh.png" width="40%">
   
   - 連接Sonata FPGA使用的杜邦線，請搭配短線使用來降低noise，否則會無法使用
   - Sonata FPGA接線可參考下圖</br>
     ![alt tag](https://i.imgur.com/TskVkC1.jpg)
 - 目前UI視窗大小固定為1078x841，若開啟後視窗無法呈現完整，需自行調整螢幕解析度或者使用外接螢幕
 - Import/export file的格式已依照使用者需求，Tool會自己轉換成可跑的Script，使用者不需再額外轉檔
   - Import file時，內建轉檔會檢查格式，格式不符就會跳出提示視窗
   - Tool產出的檔案，register’s value會自動補滿8個，但使用者自己建立file時，不需補滿
     - Example
       - ww 4001F114 DF
   - V1.03 (含)之後的版本皆有支援delay command，delay command以毫秒為單位，不支援小數點
     - Example 
       - ms 1000  &ensp;&ensp;&ensp;#表示delay 1秒
       - ms 1     &ensp;&ensp;&ensp;&ensp;&ensp;&ensp; #表示delay 1毫秒

## Structure
```
├── UI
│   ├── IC_Verification_Main.ui
│   ├── Sonata_Register_info.ui
│   ├── SPI_Cmd.ui
│   ├── File_import.ui
│   └── RG_export.ui
│
├── Script_Sonata
│   ├── Iinit_sonata.txt
│   ├── script_get_sonata_value.txt
│   ├── refresh_all.txt
│   └── func_save.txt
│
├── Sonata
│   ├── Cmd_File_New
│   │   ├── SONATA_READ_DELAY.txt
│   │   ├── SONATA_READ_WORD.txt
│   │   ├── SONATA_WRITE_DELAY.txt
│   │   └── SONATA_WRITE_WORD.txt
│   │
│   ├──protocol.py
│   └──thrd_procedure.py
│
├── image
├── Func
│   ├── func.py
│   └── spi.py
│
├── ctrl_widget.py
├── cmd_widget.py
├── Spi_cmdwidget.py
├── loadfile_widget.py
├── exportfile_widget.py
├── defaultvar.py
└── main.py
```

## Reference
 - FTDI4232H Driver安裝步驟參考文件放置在 R:\SI\Tool Team\Product Document\Sonata
 - Sonata原始資料的路徑皆已放置在 R:\SI\Tool Team\Product Document\Sonata\Original File
 - 其餘重要且整理過的資料放置在R:\SI\Tool Team\Product Document\Sonata
   - 所有register資料整理成All Register table，並請以最新日期的去參考，如有修改/新增/移除register，請另存檔案並更新日期
   - Block_diagram_Engineering_Tool JPG是放在Tool內的圖
   - 文字檔是提醒在connect/disconnect必須設定的資訊，如有修改，請記得更新
  </br></br>
 - 以上資料皆有備份到R:\SI\交接\Evelyn

## Workflow Digram
![alt tag](https://i.imgur.com/Ll7Qd0l.jpg)

## Code
### Code Explanation
|<font color="67ACF1">UI</font>|Description| 
|-|-|
|IC_Verification_Main|Main Control UI|
|Sonata_Register_info|顯示十進制與二進制register的value，並可讓使用者改值 (十進制與二進制只能擇一修改)|
|SPI_Cmd|使用者可做直接針對register做read/write的視窗|
|File_import|load file UI|
|File_export|export file UI|

</br>

|<font color="67ACF1">Script_Sonata</font>|Description| 
|-|-|
|init_sonata|設定機器連接後要做的設定值，以利register可作正常操作 (目前有8組設定)|
|script_get_sonata_value|設定所有register的address，這樣機器連接後就可以先讀取所有內部的register value|
|refresh_all|機器refresh時會觸發的script file，會重新讀取所有register value|
|func_save|K-script所需的func，基本上不用變更|

</br>

|<font color="67ACF1">Class</font>|Description| 
|-|-|
|ctrl_widget|控制main UI內所有的功能與設定|
|cmd_widget|控制Sonata Register info UI內的table呈現 (只有create, show, resize, clear table功能)|
|Spi_cmdwidget|SPI_Cmd UI的class，僅限制line edit內的輸入文字格式|
|loadfile_widget|file import UI的class|
|exportfile_widget|file export UI的class|

</br>

|<font color="67ACF1">Other</font>|Description| 
|-|-|
|ctrl_widget|控制main UI內所有的功能與設定|
|cmd_widget|控制Sonata Register info UI內的table呈現 (只有create, show, resize, clear table功能)|
|Spi_cmdwidget|SPI_Cmd UI的class，僅限制line edit內的輸入文字格式|

### Function Explanation
|<font color="67ACF1">Ctrl_widget</font>|Description| 
|-|-|
|<font color="A6DCA0">create_singal_slot</font>|button trigger 或者click後要對應連接的func設定|
|create_singal_slot|設定UI內按鈕狀態 (False = disable, True = enable)|
|closeEvent|若直接點選X關閉視窗，會觸發此func去將設定斷開，並關閉所有開啟的視窗與清除對應設定檔，避免關閉後造成下次連接時的錯誤|
|closeAllwidget|關閉所有開啟的widget|
|set_cursor_status|設定滑鼠鼠標狀態為loading|
|Information_Message|alarm window, 提示訊息可另外設定|
|clean_all_temp_file|清除所有暫存的檔案 (ex. apply或load file時，都會產生要給K-Script跑的script file，這些檔案在關閉UI後，會自動移除)|
|is_int|判斷輸入的數字是否為正整數|
|convert_hex_to_decimal|將hex list轉為decimal list|
|convert_decimal_to_hex|將decimal list轉為hex list (K-Script跑完後的資料是10進制的list)|
|get_bit_range|由於register的bit range與python list的index是相反的，所以要透過此func轉為正確的位置</br>bit range順序:&ensp;&ensp;76543210</br>python list順序: 01234567|
|get_processing_data|先將讀入的/修改的register value做前處理(轉為16進制、10進制與2進制，再逐一針對bit range)|
|set_tooltip|設定按鈕tool tip，preview各register bit range的value|
|<font color="A6DCA0">connect_to_ic</font>|與機器連接的設定func|
|clickbutton|call cmd widget，並連結對應按鈕的功能func|
|enable_tablewidget|Enable/disable widget (因為decimal table與binary table只能擇一設定)|
|display_dec_table_content|register bit range value以10進制呈現|
|display_bin_table_content|register bit range value以2進制呈現|
|on_tableWidget_itemChanged|即時改變binary table內的值 (勾選 = 1, 不勾選 = 0)|
|get_table_content|檢查table修改的內容是否與先前不同(也會檢查填寫format是否正確)，若確認有做修改，便會產生一個名稱為apply.txt的script file，並直接透過K-Script去執行寫入|
|create_apply_script_file|產出apply script|
|<font color="A6DCA0">set_SPI_cmd_window</font>|call SPI cmd widget，並連結對應按鈕的功能func|
|read_SPI_cmd|利用spi.py內的read_write func去讀取使用者想知道的register value (沒有在UI上的register也可透過此方法讀取)|
|write_SPI_cmd|利用spi.py內的write func去寫入使用者想修改的register value (沒有在UI上的register也可透過此方法修改)|
|clear_SPI_result|清空SPI cmd widget text edit 內容|
|<font color="A6DCA0">Ini_kscript</font>|lock thread，跑k-script|
|run_kscript|利用runstatus決定要做k-script的什麼事|
|ini_timer|k-script timer initial setting|
|start_timer|start k-script timer|
|stop_timer|stop k-script timer|
|timeout_timer|當跑k-script後，有回傳資料時就會進到這個func，可以利用不同的Key (saveData[0]的值)來設定要如何處理|
|<font color="A6DCA0">set_import_window</font>|call import file widget，並連結對應按鈕的功能func|
|get_import_file|確認使用者import進來的file副檔名是否為.txt，若副檔名正確就會進行轉檔動作(轉為K-script可以跑的格式)|
|convert_to_script_file|會利用func.py內的k-script func將使用者import進來的檔案作轉檔(轉檔檔案會另存在Script_Sonata folder內，關閉UI後會移除)；轉檔時會檢查import file的內容，如有錯誤的格式，就會停止轉檔，並提示使用者|
|set_import_file_to_ic|run K-script to execute the import file|
|<font color="A6DCA0">set_export_window</font>|call export file widget，並連結對應按鈕的功能func|
|export_partial_to_file|利用get_checkbox_name func回傳的dictionary去將特定名稱的register資訊存成file|
|get_checkbox_name|讀取被勾選的checkbox name，並存成dictionary|
|clear_checkbox|取消勾選被選取的checkbox|
|export_file|輸出register address and value，並存成file|
|<font color="A6DCA0">refresh_all_register</font>|run K-script 執行refresh_all的script file，重新讀取裝置內的register資訊|

</br>

|<font color="67ACF1">cmd_widget</font>|Description| 
|-|-|
|create_table_frame|建立table框架(設定row & column)|
|show_normal_table_content|顯示10進制table的內容|
|show_binary_table_content|顯示2進制table的內容|
|resize_table_frame|重新調整10進制table的框架|
|resize_bin_table_frame|重新調整2進制table的框架|
|Clear_table_content|清除表格內容|

</br>

|<font color="67ACF1">func.py</font>|Description| 
|-|-|
|set_initial_folder|建立預設export file的存放位置 (folder name是Export File(script))|
|set_string_to_list|將string轉為兩個一組的list</br>Ex. ‘01020304’→ [01, 02, 03, 04]|
|set_list_to_hexstring|將list轉為hex string|
|set_string_to_hexstring|將一般string轉為hex string|
||<font color = "F8AD58">以下是轉為K-script可用的script file語法func，詳細可參考func寫的annotation</font>|
|write_NEW_script|定義variable name|
|write_READWORD_script|定義要讀取的address (讀取register)|
|write_EQUAL_script|assign value給variable|
|write_WRITEWORD_script|將要修改的value寫到對應的address內 (寫入register)|
|write_FUNCSAVE_script|將dictionary存入list之中|
|write_SAVE_script|save function|
|write_DELAY_script|定義delay時間|


