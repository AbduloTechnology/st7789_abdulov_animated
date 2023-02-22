# st7789_abdulov_animated

Animation of Abdulo picture based upon internal FPGA resources and BlockRAM memory, not involved external resources such as memory or other interface for holding on receiving data which needs for drawing

Анша Абдуль
Биборан
Биборан
Биборан

Развивайте идеи, делайте добро, изучайте то, чему не научат в институтах

Как и обещал, исходные коды, ресурсы, материалы.

Вывод анимированного изображения выполняется только средствами FPGA на дисплей ST7789, без использования сторонних памятей, интерфейсов передачи данных, и всего остального. 

При этом, если кому-то придет в голову применить этот код, надо иметь ввиду, что данная серия дисплеев все-таки дерьмово подходит для вывода динамической картинки, так как даже на качественной сьемке видео видны искажения изображения, потому что дисплей не может быстро обновлять изображение.

Хотя, если верить даташиту, в тайминги вывода я собственно укладывался, потому что:

1. дисплей содержит 240x240 пикселей = 57600 пикселей видимой области. 
2. Командный интерфейс на каждый пиксель требует 4 слова (1 слово команды + 1 слово красный цвет + 1 слово зеленый цвет + 1 слово синий цвет) по 8 бит каждое (32 бит на один пиксель). 
3. чтобы пройти весь дисплей нужно передать 57600 пикселей по 4 байт = 230400 байт на весь экран или 1843200 бит на весь экран. 
4. Согласно даташиту, известно, что тактирование дисплею нельзя давать выше 75 МГц (по даташиту точную цифру не вспомню), так как при подаче сигнала тактирования в 100 МГц, 
можно увидеть искажения на дисплее в цвете (потому что SPI-интерфейс начинает "ехать" относительно своих же сигналов). 
5. Тактирование дисплея в моем случае было 50 МГц-тактовым сигналом от внутренней PLL в FPGA. Получается, что 50 МГц = 20 нс на период. 
6. На заполнение дисплея один раз надо потратить времени (в моем случае, исходя из вышеперечисленных параметров) 36.864.000 нс (36 мс). 
7. Это получается примерно 27 кадров в секунду должен выдавать дисплей, а он фактически выдает картинку так, как это видно на видео - то есть между изображениями видны переходы. 

Финальное изображение строится на смешении четырех слоев по определенным правилам. 

В качестве исходного изображения был выбран формат BMP, потому что он кодирует каждый пиксель, и его достаточно легко распарсить, так как не требуется использования декодера

Слои, каким цветом они рисуют изображение - формируется компонентом Layer Mixer. 

Там же я добавил возможность включить/выключить вывод того или иного слоя для поддержки какой-либо интерактивности. 

9 картинок выбрал в качестве анимации просто потому, что ожидал что не хватит внутреннего ресурса, чтобы выводить более плавное изображение. Опять же, учитывая проблему дисплея, не думаю, что выводить более плавное изображение решило бы проблему с выводом". 

Если отрезать всё до LayerMixer (память на 4 слоя + LayerMixer) и подцепить куда-нибудь на внешний интерфейс, то есть возможность грузить целые картинки с поддержкой нормальной глубины цвета. 
Так как в текущем случае двухцветность поддерживается LayerMixer-ом и самой памятью изображений, а выходной буфер (BufferRAM компонент) я заложил на возможность вывода полного 24-битного изображения. 

При всей организации на самом деле не хватает только поддержки обратной связи от интерфейса дисплея к памяти BufferRAM, в которую внешние блоки записывают в память данные. Так как при большой скорости записи в буферную память можно попасть в ситуацию, что обновление памяти происходит быстрее, чем ее чтение и отправка на непосредственно сам дисплей. 

Проект был достаточно интересный, попробовать слои смешивать между собой на аппаратном уровне оказалось достаточно простой задачей, чего не скажешь о некоторых моментах, а именно:
1. При обьявлении памяти большого размера лучше подключать файл со значениями памяти
2. Парсить картинки через TCL оказалось не лучшим решением в плане скорости, однако позволяет прочувствовать основы tcl-скриптов, работу ввода-вывода в TCL, и всякое разное. 

Данный код писался под видео. Совместив репозитории st7789_abdulov_static и данный репозиторий можно понять маршрут проектирования подобных вещей и подобных устройств. 
На видео я старался всё по делу обьяснить, без растягивания рассказа о том, как классно запускать Vivado и какая кнопочка за что отвечает. 

Ресурс утилизации BRAM для данного проекта - 100% (75 BRAM, если мне не изменяет память). Буфер при этом сжирает больше всего ресурса. 

Во всяком случае цель данной работы была следующая :
0. Показать, что есть такое движение как Абдуловера, которое достаточно многообразно, и занимается многими вопросами различного характера, в том числе и технологии FPGA, и многого другого.
1. Познакомить людей с таким классом устройств как FPGA
2. На наглядном примере показать маршрут проектирования каких-либо более менее сложных функциональных блоков
3. Сопустствующая работа, основные шаги и моменты. 
4. Практические результаты, полученный опыт. 
