> Niektóre polecenia nie są obsługiwane, jeśli aplikacja korzysta z bazy danych SQLite jako magazynu danych tożsamości. Ze względu na ograniczenia w aparacie bazy danych `Alter` polecenia throw następujący wyjątek:
>
> "System.NotSupportedException: SQLite nie obsługuje tej operacji migracji." 
>
> Jako obejście Uruchom migracje Code First na bazę danych do tabel zmian.
