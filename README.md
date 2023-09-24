There are the tables used in the analysis of the data, below a the details about the tables and columns present.<br/>
tables : 5<br/>
card_holder : id, name<br/>
credit_card: card, id_card_holder<br/>
merchant: id, name, id_merchant_category<br/>
merchant_category: id, name<br/>
transactions: id, date, amount, card, id_merchant<br/>

Note: 
1. credit_card and credit card holder have a foreign key : id_card_holder and id
2. merchant and merchant_category have a foreign key : id_merchant_category and id
3. transaction and merchant have a foreign key : id_merchant and id
4. transaction and credit_card have a foreign key : card and card

Enjoy your time analyzing this dataset.

Toodles!
