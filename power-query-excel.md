# Abfrage Sascha
```code
let
    Quelle = Excel.CurrentWorkbook(){[Name="Tabelle1"]}[Content],
    // Schritte zur Klassifizierung
    #"Kategorie erstellen" = Table.AddColumn(Quelle, "Kategorie", each
        if Text.Contains([#"Auftraggeber/Empfänger"], "NORMA")  
        or Text.Contains([#"Auftraggeber/Empfänger"], "SCHERF")
        or Text.Contains([#"Auftraggeber/Empfänger"], "HERKULES")
        or Text.Contains([#"Auftraggeber/Empfänger"], "Herkules") 
        or Text.Contains([#"Auftraggeber/Empfänger"], "REWE")
        or Text.Contains([#"Auftraggeber/Empfänger"], "LIDL") 
        or Text.Contains([#"Auftraggeber/Empfänger"], "SG ")
        or Text.Contains([#"Auftraggeber/Empfänger"], "HELBING")
        or Text.Contains([#"Auftraggeber/Empfänger"], "KAUFLAND")
        or Text.Contains([#"Auftraggeber/Empfänger"], "METRO")
        or Text.Contains([#"Auftraggeber/Empfänger"], "FLEISCHEREI")
        or Text.Contains([#"Auftraggeber/Empfänger"], "NAHRSTEDT")
        or Text.Contains([#"Auftraggeber/Empfänger"], "SCHAEFERS")     
        or Text.Contains([#"Auftraggeber/Empfänger"], "GLOBUS")
        or Text.Contains([#"Auftraggeber/Empfänger"], "BAECKER")
        or Text.Contains([#"Auftraggeber/Empfänger"], "NETTO")
        or Text.Contains([#"Auftraggeber/Empfänger"], "Netto")
        or Text.Contains([#"Auftraggeber/Empfänger"], "EDEKA")
        or Text.Contains([#"Auftraggeber/Empfänger"], "ALDI")
        or Text.Contains([#"Auftraggeber/Empfänger"], "real") 
        or Text.Contains([#"Auftraggeber/Empfänger"], "Schafers")         
        then "Lebensmittel"
        
        else if Text.Contains([Verwendungszweck], "AMZN") or Text.Contains([Verwendungszweck], "Amazon") or Text.Contains([#"Auftraggeber/Empfänger"], "AMZN") or Text.Contains([#"Auftraggeber/Empfänger"], "AMAZON") then "Amazon"      
        else if Text.Contains([Verwendungszweck], "VERSICHERUNG") then "Versicherung"    
        else if Text.Contains([#"Auftraggeber/Empfänger"], "APOTHEKE") or Text.Contains([#"Auftraggeber/Empfänger"], "ROSSMANN") 
        or Text.Contains([#"Auftraggeber/Empfänger"], "Drogerie") 
        or Text.Contains([#"Auftraggeber/Empfänger"], "H+M")  
        or Text.Contains([#"Auftraggeber/Empfänger"], "KIK")
        or Text.Contains([#"Auftraggeber/Empfänger"], "IKEA")
        or Text.Contains([#"Auftraggeber/Empfänger"], "ERNSTING")
        or Text.Contains([#"Auftraggeber/Empfänger"], "HOEFFNER")
        or Text.Contains([#"Auftraggeber/Empfänger"], "Baumarkt")
        or Text.Contains([#"Auftraggeber/Empfänger"], "T PHILIPPS")
        or Text.Contains([#"Auftraggeber/Empfänger"], "BAUMARKT")
        or Text.Contains([#"Auftraggeber/Empfänger"], "HELLWEG")
        or Text.Contains([#"Auftraggeber/Empfänger"], "HUGENDUBEL")
        or Text.Contains([Verwendungszweck], "GIROCARD")
        then "Allgemein/Non-Food"

        else if Text.Contains([#"Verwendungszweck"], "Wohngeld")  then "Wohngeld"
        else if Text.Contains([#"Verwendungszweck"], "WP-ABRECHNUNG")  then "Wertpapiere"
        else if Text.Contains([#"Auftraggeber/Empfänger"], "Bundeskasse - Dienstort Weiden -") or Text.Contains([#"Auftraggeber/Empfänger"], "Bundeskasse - Dienstort Trier -") then "Gehalt"
        else if Text.Contains([#"Auftraggeber/Empfänger"], "Drillisch Online GmbH") then "Handy"
        else if Text.Contains([#"Verwendungszweck"], "Kfz-Steuer") or Text.Contains([#"Auftraggeber/Empfänger"], "VISA OIL 427") then "Auto"
        else if Text.Contains([#"Auftraggeber/Empfänger"], "LBS HESSEN-THUERINGEN") then "Bausparer"
        else if Text.Contains([#"Auftraggeber/Empfänger"], "Bargeldauszahlung") then "Allgemein"
        else null
    ),

    // Optional: Filtern und Entfernen unnötiger Spalten
    #"Nur Kategorien anzeigen" = Table.SelectRows(#"Kategorie erstellen", each [Kategorie] = null),
    #"Unnötige Spalten entfernen" = Table.RemoveColumns(#"Nur Kategorien anzeigen", {"Verwendungszweck"})
in

    //#"Nur Kategorien anzeigen"
    #"Kategorie erstellen"
```code
# Abfrage Gemeinschaft
