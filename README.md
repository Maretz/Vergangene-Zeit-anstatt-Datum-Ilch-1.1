In diesem Tutorial geht es um die Zeitangabe in den Beiträgen im Forum. Anstatt dem Datum wird die vergangene Zeit angezeigt.
Als Grundlage werden hier den Original Ilch Forum Dateien genutzt. Allerdings sollten sich diese nicht viel ändern zu den jeweilgen Mods bis auf die Zeilenangaben.

Ich gehe mal davon aus, dass gewisse Grundkenntnisse vorhanden sind, u.a. der richtige Editor genutzt wird. ;)
Als erstes öffnen wir die include/contents/forum/show_posts.php .
Oberhalb der Datei fügen wir als erstes den Ländercode ein. Dies erspart uns die Arbeit die Wochentage sowie Monatsnamen in Deutsch umzuwandeln.

PHP-Quellcode
```ruby
    setlocale(LC_TIME, "de_DE");
```


Natürlich können andere Ländercode eingegeben werden, anstatt de_DE .

In Zeile 56 siehst du den bestehenden Zeitwiedergabe:

PHP-Quellcode
```ruby
    	$row['date'] = date ('d.m.Y - H:i:s', $row['time'] );
```


Diesen ersetzen wird mit folgenden Codesatz :

PHP-Quellcode
```ruby
    # Timestramp aus der Datenbank
    $times = $row['time'];
    # Differenz errechnen aus aktueller Zeit und Beitrags-Zeit
    $diff = time() - $row['time']; 
    $fullHours = intval($diff/60/60); 
    $Minutes = intval(($diff/60)-(60*$fullHours));
    # Minuten 
    if ($Minutes == 0) {
    $Minutes = 'vor einem Moment';
    } elseif ($Minutes == 1) {
    $Minutes = 'vor einer Minute';
    } else {
    $Minutes = 'vor '. $Minutes .' Minuten';
    }
    # Stunden, ab 59 Minuten Stunden mit Uhrzeit  
    $zeitposts = strftime("(%H:%M Uhr)", $times);
    if ($fullHours == 0) {
    $Stunde = $Minutes;
    } elseif ($fullHours == 1) {
    $Stunde = 'vor einer Stunde '. $zeitposts;
    } else {
    $Stunde = 'vor '. $fullHours .' Stunden '. $zeitposts;
    } 
    # gibt den Wochentag aus
    $wochentag = strftime("%A", $times); 
    if (date("d.m.Y", $times) == date("d.m.Y")) {
    			# Innerhalb des restlichen Tages - Bis 12 Stunden, danach Heute, H:M Uhr
    			if ($fullHours < 12) {
    				$row['date'] = $Stunde;
    			} else {
    				$row['date'] = strftime("Heute, %H:%M Uhr", $times);
    			}
    		} elseif (date("d.m.Y", $times) == date("d.m.Y", time() - 60 * 60 * 24)) {
    			# Innerhalb 24h mit Tagüberschreitung - Bis 12 Stunden, danach Gestern, H:M Uhr
    			if ($fullHours < 12) {
    				$row['date'] = $Stunde;
    			} else {
    				$row['date'] = strftime("Gestern, %H:%M Uhr", $times);
    			}
    		   # Innerhalb 48h - Wochentag, H:M Uhr
    		} elseif (date("d.m.Y", $times) == date("d.m.Y", time() - 60 * 60 * 48)) {
    			$row['date'] = "$wochentag, " . date("H:i", $times) . " Uhr";
    		   # Innerhalb 72h - Wochentag, H:M Uhr
    		} elseif (date("d.m.Y", $times) == date("d.m.Y", time() - 60 * 60 * 72)) {
    			$row['date'] = "$wochentag, " . date("H:i", $times) . " Uhr";
    		   # Innerhalb 96h - Wochentag, H:M Uhr
    		} elseif (date("d.m.Y", $times) == date("d.m.Y", time() - 60 * 60 * 96)) {
    			$row['date'] = "$wochentag, " . date("H:i", $times) . " Uhr";
    		   # Innerhalb 120h - Wochentag, H:M Uhr
    		} elseif (date("d.m.Y", $times) == date("d.m.Y", time() - 60 * 60 * 120)) {
    			$row['date'] = "$wochentag, " . date("H:i", $times) . " Uhr";
    		   # Nach 120h - Datum mit Uhrzeit
    		} else {
    			$row['date'] = strftime("%d. %B %Y - %H:%M Uhr", $times);
    		}
```


Den Codesatz habe ich bereits Kommentiert um die Bereiche besser überblicken zu können.
Wie man am Anfang sieht beziehen wir auch hier erstmal den Timestramp vom Beitrag aus der Datenbank. Als nächstes benötigen wir die Differnz aus der Beitrags-Zeit und der Aktuellen.
Dabei wird einfach die Beitragszeit von der aktuellen abgezogen (Timestramp) .

Als nächstes wird die Differnz in Minuten / Stunden umgewandelt. ( PHP Gleichungen )
Stunden = differenz / 60 / 60
Minuten = (differenz / 60)-(60 * Stunden)

Darauf gehe ich mal nicht näher ein, da es dazu schon recht viele Themen gibt. :) Jetzt haben wir schonmal die Basis.
Jetzt wollen wir, dass die erste Minute nicht als 0 erscheint, sondern mit "vor einem Moment" . Dies wird mit einer if Anweisung umgesetzt. Die Bedingungen sind ja klar.
$Minutes == 0 ergibt vor einem Moment , $Minutes == 1 ergibt vor einer Minute. Ein weiterer Eintrag ohne Bedingungen die restliche Zeit.

So verhält sich das auch mit den Stunden.
Mit dem Unterschied das bei 0 Stunden die Minuten angezeigt werden. Ab $fullHours == 1 werden die Stunden angezeigt
Allerdings habe ich hier noch die Uhrzeit angehängt . Den eine Stunde ist lang und da können einige Beiträge aufkommen.

Den Wochentag haben wir ja einfach über Ländercode erworben.
Ok, kommen wir zu nächsten Teil. In diesem wird bestimmt, was in welchem Zeitraum angezeigt wird. An erster Stelle geht es um den aktuellen Tag. In diesem Codesatz werden die Stunden bis max. 11 Stunden angezeigt, danach wird Heute mit der Uhrzeit angezeigt. Ich finde es so besser, anstatt z.b . vor 23 Stunden, wenn jemand morgens um 0.01 Uhr einen Beitrag erstellt. :) . Der Wert kann natürlich geändert werden.
Genauso verhält es sich am Tag darauf. Allerdings mit Anzeige Gestern, Uhrzeit .. .
Sollte jemand um 22.00 Uhr einen Beitrag verfassen, werden auch in diesem Fall die bis max. 11 Stunden angezeigt. Sonst würde ja nach 2 Stunden schon Gestern, ... angezeigt.

Ab 48 Stunden werden dann die Wochentage mit Uhrzeit angezeigt, bis max. 120 Stunden danach. Drüber hinaus würde ich nicht gehen. Es wäre verwirrend, wenn Samstag jemand einen Beitrag liest mit der Zeitangabe Samstag, Uhrzeit.... , obwohl dieser schon eine Woche besteht... :D
Diese Bereiche definieren einen Tag, also nicht die vergangene Zeit. 24 h ist ein Tag , 48 2 Tage usw. .
Es sollte jeder 24h Bereich ( 24/48/ ...)
in einer Bedingung stehen, da sonst die letzte Anweisung in Kraft tritt.
120 h sind dann 6 Tage. Wem das zu lange ist mit der Wochentag Anzeige, der kann auch einfach die 120h Bedingung löschen.
Nach 120 Stunden steht dann das Datum mit Uhrzeit da. Dabei ist der Monat voll ausgeschrieben.

FAZIT:

Feine Sache. Zudem doch sicherlich besser zur Übersicht der aktuellen Beiträge.
Dies lässt sich auf so jeden Bereich ( News, Forum, PM System ... ) umsetzen.
Bei den News im Ilch CMS muss man allerdings erst die Zeit in Timestramp umwandeln, da diese in einem anderen Format in der Datenbank gespeichert wird.

PHP-Quellcode
```ruby
    $newstimestramp = new DateTime($row['news_time']);
    	$row['timesnews'] = $newstimestramp->getTimestamp();
```


$row['timesnews'] wäre dann der Timestramp der News.