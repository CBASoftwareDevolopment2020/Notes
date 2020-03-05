# [Relational Databases Recap](https://datsoftlyngby.github.io/soft2020spring/DB/week-06/#1-relational-databases-recap)

06-02-2020

|      |                                            |
| ---- | ------------------------------------------ |
| DBMS | **D**ata**B**ase **M**anegment **S**ystems |
| SQL  | **S**tructured **Q**uery **L**anguage      |
| DDL  | **D**ata **D**efinition **L**anguage       |
| DML  | **D**ata **M**anipulation **L**anguage     |

-   Funktionel Afhængighed: En kolonnes værdi er afhængig af værdien i en anden kolonne.
-   Transitiv Afhængighed: Den kolonne man er funktionelt afhængig af, er funktionelt afhængig af en anden kolonne.

## Normalformer

Normalisering sker i det logiske niveau

-   Første Normalform
    -   Skal have en primary key
-   Anden Normalform
    -   En primary key må ikke være funktionelt afhængig
-   Tredje Normalform
    -   Deler den store tabel op i mindre tabeller

## Join

Join er et subset af det kartianske produkt.  
A JOIN B ⊂ A X B  
[Relational Algebra](https://slideplayer.com/slide/8666326/)
![[Relational Algebra](https://slideplayer.com/slide/8666326/) slide 4](https://images.slideplayer.com/26/8666326/slides/slide_4.jpg)
![[Relational Algebra](https://slideplayer.com/slide/8666326/) slide 5](https://images.slideplayer.com/26/8666326/slides/slide_5.jpg)

## [ACID](https://premaseem.wordpress.com/tag/acid-properties/)

![ACID](https://slideplayer.com/slide/9307681/28/images/4/Transaction+ACID+Properties.jpg)

-   **A**tomicity
    -   Alt eller intet
-   **C**onsistency
    -   Kun valid data bliver gemt
-   **I**solation
    -   Transaktioner påvirker ikke hinanden
-   **D**urability
    -   Gemt data bliver ikke tabt
