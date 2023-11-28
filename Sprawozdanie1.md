## Projektowanie Usług w Chmurze 
### Ćwiczenie: Baza danych MS SQL Server w Azure
Wojciech Zieliński
305 245

---
## Baza danych MS SQL Server w Azure
#### Krok 1: Utworzenie konta w Azure.
W tym celu wykorzystano konto z domeną politechniki, na którym dodatkowo aktywowano darmową subskrypcję studencką. 

#### Krok 2: Utworzenie bazy danych.
Baza danych wraz z odpowiednimi zasobami została utworzona w oparciu o instrukcję do ćwiczenia oraz tutoriale dostępne na platformie YouTube. W trakcie tworzenia bazy danych skorzystano z przykładowej bazy.

#### Krok 3: Połączenie z bazą danych.
W tym celu z poziomu strony głównej bazy danych, należało przejśc do przeglądu (1) oraz parametrów połączeń (2).  
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/03c1f549-12ef-4ee6-abb1-45a7f732f6ef)


Następnie należało skopiować zaznaczony na poniższym zdjęciu fragment oraz wkleić go w rubryce "Server name" w aplikacji SQL Server Management Studio oraz zalogować się podając dane admina.

![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/80370780-860e-4f31-bf75-1581bf697eed)
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/d3212031-2eb7-4e60-b2b1-f74a1599b563)

Kolejne kroki uruchomienia maszyny wirtualnej oraz prazcy z Azure Table Storage nie powiodły się.


#### Krok 4: Konfiguracja Firewalla Azure SQL Database

W celu założenia firewalla należało wybrać serwer,, przejść do ustawień oraz podać adres IP klienta, który może mieć dostęp do danych. W tym celu podano swój obecny adres IP.
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/4147db35-3353-4200-b37b-007bab6a0db5)

---

## Azure Cosmos DB
W pierwszej kolejności należało stworzyć konto w Azure Cosmos DB. Podczas tworzenia konta był wybór między różnymi API, ja zdecydowałem się na pierwszą opcję, Azure Comsos DB for No SQL.
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/81f04715-0966-4693-89d4-5e5d3f6a4a73)

Następnie dokonano odpowiedniej konfiguracji
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/d77857b6-a21a-4265-bf14-326f03da09ce)
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/c12e978a-398d-4fe8-8a4e-4cdb6377847e)

Potem utworzono kontener
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/b7c61394-66d0-4ed6-859d-e81860d740ed)
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/f82b221c-1f33-4772-8ac7-6cc9a235b44b)


Następnie próbowano w języku Python utworzyć podstawowe operacje CRUD na bazie danych, jednak wystąpiły komplikacje podczas próby łączenia aplikacji z bazą. 
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/6ed414e2-06a7-48b8-9214-0c5e7c4ddd26)
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/82def97c-684a-43cf-9695-4b89961996c4)

Spróbowano również wykonać operacje za pomocą graficznego interfejsu Azure i tu już nie pojawiły się żadne problemy.
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/dc6f61ae-0d43-459f-a433-cec2205d771e)
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/cff7f31c-f12a-4307-a143-7bd75ce09c6c)

Nie udało się jednak zmienić liczby jednostek przetwarzania żądań ponieważ minimalna wartość to 1000 RU/s, a większe wartości nie są dostępne.
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/a46e1764-df28-4532-81a5-1c3c66d79042)
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/85fa1b79-a7c5-4444-b724-20a1f770ad22)

