Классы для выгрузки поьзовательских данных из SAP R/3 в шаблон DOCX документа и выгрузки шаблона на сервер, используя стандартные классы SAP для работы с DOCX документами.
Базовый класс ZCL_DOCX_FORM является копией стандартного класса CL_DOCX_DOCUMENT. 
Начиная с Microsoft Word 2007, когда вы создаете новый документ в Word и сохраняете его, создается новый файл с расширением "* .docx".
Этот файл представляет собой сжатые XML-файлы, которые описывают весь документ Word целиком. Он включает тексты, таблицы, размеры шрифтов, цвета, комментарии, настройки полей, настройки разделов и все, 
что пользователь вручную разместил и поддерживал в документе. Речь идет о XML-файлах, связанных посредством связей друг с другом в определенной структуре 
и заархивированных в файл.
Метод заполнения шаблона документа DOCX пользовательскими данными заключается в том, что из xml-файлов, описывающих документ DOCX через методы стандартного класса CL_DOCX_DOCUMENT достаются xml файлы, 
описывающие определенную часть данного документа, затем через стандартные функциональные модули из xstring xml переводятся в формат string. Затем через REPLACE метки в xml части, переведенной в формат string, заменяются на пользовательские данные.
Для заполнения колонтитула и основной части документа пользовательскими данными созданы два класса ZCL_DOCX_HEADER и ZCL_DOCX_MAIN соответственно.
Для заполнения таблиц в DOCX документе создан отдельный класс, так как заполнение таблиц осуществляется по другому.

Для динамического заполнения таблиц требуется провести небольшие манипуляции со стандартным классом CL_DOCX_DOCUMENT.
Класс ZCL_DOCX_FORM является копией класса CL_DOCX_FORM. После копирования класса вам необходимо исправить 3 ошибки, из-за которых не удается активировать только что созданный Z-класс. Вы быстро их найдете, вам просто нужно переписать "cl_docx_form" на "zcl_docx". Затем перейдите к методу MERGE_DATA, где вам нужно внести 2 небольших изменения:

1. Найдите и перепишите следующий код, в котором создается имя преобразования:
  CONCATENATE 'XSLT' ls_xsltattr-xsltdesc INTO ls_xsltattr-xsltdesc.
например, для этого:
  CONCATENATE 'Z' ls_xsltattr-xsltdesc INTO ls_xsltattr-xsltdesc.
Причина этого изменения в том, что вам разрешено создавать преобразования только в пространстве имен Z или customer, а не в пространстве имен XSLT.
2. Найдите функциональный модуль XSLT_MAINTENANCE для удаления сгенерированного преобразования и вставьте входной параметр i_gen_flag со значением abap_true
![image](https://github.com/user-attachments/assets/fd7f0e19-9e2a-4c7d-a817-4c1d9181cc3c)

  Причина этого изменения заключается в том, чтобы избежать всплывающего окна во время выполнения (например, если оно используется в WebUI).
  ZCL_DOCX_FORM готов к использованию. Теперь следует подготовить шаблон Word документа.
  1. Следует подготовить пользовательский XML-файл.
  Например:
![image](https://github.com/user-attachments/assets/bee349e2-a15c-4755-bf75-51ff342c1667)

Убедитесь, что пространство имен тега data точно соответствует ""http://www.sap.com/SAPForm/0.5"". Эта проверка используется в упомянутом методе merge_data. Наша таблица представлена таблицей узлов, и каждая строка в этой таблице представлена тегом <DATA>...</DATA>. Теги внутри раздела <DATA> - это просто столбцы в таблицах.
Создайте новый файл docx под названием Table.docx.  Вставьте в документ простую таблицу с названием строки заголовка, фамилией и датой рождения и одной пустой строкой. Теперь отметьте эту пустую строку и нажмите кнопку, повторяющую управление содержимым раздела. Ваш документ Word должен выглядеть следующим образом:
![358841134-d2f31aa1-fd0b-4aab-b494-14c106167da3](https://github.com/user-attachments/assets/138c4713-a6c1-4a1b-b97a-68190a55553e)
Теперь прикрепите пользовательский XML-файл, созданный на предыдущем шаге, и сопоставьте его атрибуты столбцам. Результат должен выглядеть следующим образом:
![image](https://github.com/user-attachments/assets/883e5a6e-896f-4dab-b2f4-20436c80d94c)
Если у Вас возникнут проблемы, то вы можете обратиться к этому блогу: https://community.sap.com/t5/application-development-blog-posts/openxml-in-word-processing-custom-xml-part-mapping-flat-data/ba-p/13331900.
Для работы с таблицами используется класс ZCL_DOCX_TABLE.
Для автоматизации заполнения каждой отдельной части шаблона  документа пользовательскими данными используются классы ZCL_DOCX_MAIN, ZCL_DOCX_HEADER и ZCL_DOCX_TABLE. Данные классы наследуются от базового класса ZCL_DOCX_FORM, который в свою очередь наследуется от стандартного класса CL_OPENXML_PACKAGE. Методом, через который начинается заполнение шаблона документа DOCX пользовательскими данными является метод OPENFORM. На вход в данный метод подается таблица типа ZHR_DOCX_VALUE_T.
Описание типа таблицы из ABAP словаря представлено на скриншоте ниже:
![image](https://github.com/user-attachments/assets/9b5c2072-9177-45c8-a1b6-7fabb88d6eb5)
![image](https://github.com/user-attachments/assets/23aa8045-9a86-46ff-926d-71ef9cff3b33)
Для передачи названия таблицы и значения, которое будет заменять метку в шаблоне документа DOCX создан домен и элемент данных ZSTRING.
Описание домена и элемента данных из ABAP словаря представлено на скриншоте ниже:
![image](https://github.com/user-attachments/assets/5a613c1a-03fd-43dc-9b8a-88850611484c)
![image](https://github.com/user-attachments/assets/2293afbc-7abd-425e-9231-b6fb2ac5d63f)
![image](https://github.com/user-attachments/assets/a7afe41b-466d-4b3c-9c8f-d6f2582cc456)
Пример заполнения шаблона документа DOCX пользовательскими данными представлен в демонстрационной программе zhr_tyarasov_zcldocx_test, которая находится в данном репозитории.



