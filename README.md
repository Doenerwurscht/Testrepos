# Testrepos

# Create Table
In diesem Skript wird die Datenbank nach Vorlage des ERM in der richtigen Reihenfolge erstellt. 
## ERM
Folgendes ERM ist Grundlage der MariaDB:  

![ERM Fluggesellschaften](https://github.com/Doenerwurscht/Testrepos/blob/main/1.2%20Fluggesellschaften.png)
 
## Ebene 1

Im ersten Abschnitt werden alle Entitätstypen erstellt die keine _zu-1-Beziehung_ zu einem anderen Entitätstyp haben.  
Hierbei handelt es sich um:  
*   die Fluggesellschaften  
*   die Flughäfen
*   die Reisebüros
*   die Passagiere
```MySql
CREATE TABLE	Fluggesellschaft	(Kurzbezeichnung varchar (2) not null,
					Name varchar (100),
					Hauptsitz varchar(100),
					primary key (Kurzbezeichnung));

CREATE TABLE	Flughafen		(FlughCode varchar(3) not null,
					Name varchar(100),
                                        AnzahlPisten decimal (2),
                                        primary key (FlughCode));

CREATE TABLE	Reisebuero		(BueroName varchar (100) not null,
					Strasse varchar (100),
					Hausnummer varchar(5),
					Postleitzahl decimal (5) not null,
                                        Ort varchar(100),
                                        Groesse decimal (4),
                                        Mitarbeiterzahl decimal (4),
                                        primary key (BueroName,Postleitzahl));

CREATE TABLE	Passagier		(Vorname varchar (100) not null,
					Nachname varchar (100) not null,
                                        Strasse varchar (100),
					Hausnummer varchar(5),
					Postleitzahl decimal (5) not null,
                                	Ort varchar(100),
                                        Geschlecht varchar (15),
                                        primary key (Vorname, Nachname, Strasse, Hausnummer, Postleitzahl, Ort));

```
## Ebene 2
```MySql                                    
CREATE TABLE	Flugzeug		(IdentNr decimal (4) not null,
					Typ varchar (100),
                                    	Fluggesellschaft_Kurzbezeichnung varchar (2) not null,
                                    	primary key (IdentNr, Fluggesellschaft_Kurzbezeichnung),
                                    	foreign key (Fluggesellschaft_Kurzbezeichnung) references Fluggesellschaft(Kurzbezeichnung));

CREATE TABLE	Flugticket		(TicketNr decimal (10) not null,
					Datum date,
                                    	Preis decimal (12,2),
                                    	Reisebuero_BueroName varchar (100) not null,
                                    	Reisebuero_Postleitzahl decimal (5) not null,
                                    	Passagier_Vorname varchar (100) not null,
					Passagier_Nachname varchar (100) not null,
					Passagier_Strasse varchar (100),
                                    	Passagier_Hausnummer varchar(5),
					Passagier_Postleitzahl decimal (5) not null,
                                    	Passagier_Ort varchar(100),
                                    	primary key (TicketNr, Reisebuero_BueroName, Reisebuero_Postleitzahl),
                                    	foreign key (Reisebuero_BueroName, Reisebuero_Postleitzahl) references Reisebuero(BueroName, Postleitzahl),
                                    	foreign key (Passagier_Vorname, Passagier_Nachname, Passagier_Strasse, Passagier_Hausnummer, Passagier_Postleitzahl, Passagier_Ort)
                                    	references Passagier(Vorname, Nachname, Strasse, Hausnummer, Postleitzahl, Ort));

CREATE TABLE	Linienflugangebot	(AngebotsNr decimal (10) not null,
					SollAbflugzeit time,
                                    	Fluggesellschaft_Kurzbezeichnung varchar (2) not null,
                                    	Start_Flughafen_FlughCode varchar(3) not null,
                                    	Ziel_Flughafen_FlughCode varchar(3) not null,
                                    	primary key (AngebotsNr, Fluggesellschaft_Kurzbezeichnung),
                                    	foreign key (Fluggesellschaft_Kurzbezeichnung) references Fluggesellschaft(Kurzbezeichnung),
                                    	foreign key (Start_Flughafen_FlughCode) references Flughafen(FlughCode),
                                    	foreign key (Ziel_Flughafen_FlughCode) references Flughafen(FlughCode));
```                                    
## Ebene 3
```MySql 
CREATE TABLE	Flugschein		(FlugschNr decimal (10) not null,
					Flugklasse varchar (2),
                                    	Flugticket_TicketNr decimal (10) not null,
					Reisebuero_BueroName varchar (100) not null,
                                    	Reisebuero_Postleitzahl decimal (5) not null,
                                    	Passagier_Vorname varchar (100) not null,
					Passagier_Nachname varchar (100) not null,
					Passagier_Strasse varchar (100),
                                    	Passagier_Hausnummer varchar(5),
					Passagier_Postleitzahl decimal (5) not null,
                                    	Passagier_Ort varchar(100),
					primary key (FlugschNr, Flugticket_TicketNr, Reisebuero_BueroName, Reisebuero_Postleitzahl),
                                    	foreign key (Flugticket_TicketNr) references Flugticket(TicketNr),
                                    	foreign key (Reisebuero_BueroName, Reisebuero_Postleitzahl) references Reisebuero(BueroName, Postleitzahl),
                                    	foreign key (Passagier_Vorname, Passagier_Nachname, Passagier_Strasse, Passagier_Hausnummer, Passagier_Postleitzahl, Passagier_Ort)
                                    	references Passagier(Vorname, Nachname, Strasse, Hausnummer, Postleitzahl, Ort));
```                                    
## Ebene 4
```MySql 
CREATE TABLE	Flug	            	(Flugtag date not null,
				    	IstAbflugzeit time,
                                    	Linienflugangebot_AngebotsNr decimal (10) not null,
                                    	Fluggesellschaft_Kurzbezeichnung varchar (2) not null,
                                    	Flugzeug_IdentNr decimal (4) not null,
				    	Flugschein_FlugschNr decimal (10) not null,
                                    	primary key (Flugtag, Linienflugangebot_AngebotsNr, Fluggesellschaft_Kurzbezeichnung),
                                    	foreign key (Linienflugangebot_AngebotsNr) references Linienflugangebot(AngebotsNr),
                                    	foreign key (Fluggesellschaft_Kurzbezeichnung) references Fluggesellschaft(Kurzbezeichnung),
                                    	foreign key (Flugzeug_IdentNr) references Flugzeug(IdentNr),
                                    	foreign key (Flugschein_FlugschNr) references Flugschein(FlugschNr));
                                    
```
