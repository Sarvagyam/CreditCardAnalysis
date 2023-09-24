There are the tables used in the analysis of the data, below a the details about the tables and columns present.
tables : 5
card_holder : id, name
credit_card: card, id_card_holder
merchant: id, name, id_merchant_category
merchant_category: id, name
transactions: id, date, amount, card, id_merchant

Note: 
1. credit_card and credit card holder have a foreign key : id_card_holder and id
2. merchant and merchant_category have a foreign key : id_merchant_category and id
3. transaction and merchant have a foreign key : id_merchant and id
4. transaction and credit_card have a foreign key : card and card

Enjoy your time analyzing this dataset.

Toodles!
