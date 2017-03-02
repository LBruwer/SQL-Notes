# SQL quick Reference

Contents:

- [ARPU](#ARPU)
- [6 Months Back](#6 Months Back)









## Code Bits

<a name="ARPU"/>

### ARPU
/* Customers ARPU spending > R1000 in the last 6 months */

-- JHBBI-DW  DW 

with ARPU
as
(
select iCoId
      ,ARPU = sum(mCtAmount) / 6
from fac.ContractInvoice
where dCtTuDate >= dateadd(mm, -6, (select max(dCtTuDate) from fac.ContractInvoice))
group by iCoId
)
select *
from ARPU a
  join fac.ContractDetail cd
    on cd.iCoId = a.iCoId
  join fac.CustomerProfiling cp
    on cd.iCoId = cp.iCoId
where a.ARPU >= 1000 and cd.sStatus in ('a','s') and cp.sSegments = 'Consumer'
-----------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------------------------

<a name="6 Months Back"/>

### 6 Months Back
Eerstens, om die begin van die huidige maand te kry: 
 
	select dateadd(mm, datediff(mm, 0, getdate()), 0) 
 
Wat die kode hierbo doen is dit kry die verskil in maande 
vanaf 0 tot vandag se datum. Daarna add dit daardie hoeveelheid 
maande weer terug by 0. Dit gee die eerste dag van die huidige maand. 
As jy 0 ingee gebruik SQL se interne default begin datum om die berekening te doen. 
Jy kan iets soos ‘1900-01-01’ ook gebruik, maar hierdie is veiliger in meeste gevalle.
 
Tweedens gaan ons 6 maande aftrek:
 
	select dateadd(mm, datediff(mm, 0, getdate())-6, 0)
 
Dit gee hierdie maand dan nou 2014-03-01 en volgende maand 
sal dit 2014-04-01 gee en so aan. In plaas daarvan om die hoeveelheid 
maande net so by 0 te tel trek ons eers 6 af.
 
Laastens convert ons dit dan na ‘n datum formaat wat pas by die een
in die table en gooi als behalwe die eerste 6 karakters weg:
 
	select convert(varchar(6), dateadd(mm, datediff(mm, 0, getdate())-6, 0), 112)
 
Dit bring die 201403 terug wat jy soek. Dan verander ons die vaste “201402”
van jou query met die expression hierbo:
 
	 
	SELECT 
	       [COID]
	      ,[Period]
	      ,[ARPU]
	      ,[CTAmount]
	      ,[Retention]
	      ,[ServicedAutopage]
	      ,[ServicedPartner]
	      ,RET_CHL1 = Ret.sCustomerLevel1
	      ,RET_CHL2 = Ret.sCustomerLevel2
	      ,RET_DEALER = Ret.sDealerName
	      ,RET_DEALERC = Ret.sDealercode
	      ,ACQ_CHL1 = Acq.sCustomerLevel1
	      ,ACQ_CHL2 = Acq.sCustomerLevel2
	      ,ACQ_DEALER = Acq.sDealerName
	      ,ACQ_DEALERC = Acq.sDealercode
	  FROM [Comms].[ContractElements]
	 
	  LEFT JOIN DW.dim.PartnerListNew as Acq ON Acq.iDealerID = Comms.ContractElements.DealerID 
	  LEFT JOIN DW.dim.PartnerListNew as Ret ON Ret.iDealerID = Comms.ContractElements.RETDealerID 
	 
	  WHERE ([Comms].[ContractElements].[Period] > convert(varchar(6), dateadd(mm, datediff(mm, 0, getdate())-6, 0), 112) ) and (Retention = 1)




