with bp as (SELECT *
from p1_cdw_ad..business_partner b
inner join p1_cdw_ad..country cu on cu.country_id=b.country_id
inner join p1_cdw_ad..corporate_brand c on c.corporate_brand_id=b.corporate_brand_id
WHERE DATE(MEMBER_CREATION_DATETIME) <= CURRENT_DATE - INTERVAL '1 DAY' and DATE(MEMBER_CREATION_DATETIME) >= CURRENT_DATE - INTERVAL '10 DAY'
and CUSTOMER_DATA_ORIGIN_ID not in (select CUSTOMER_DATA_ORIGIN_ID from p1_cdw_ad..customer_data_origin
where CUSTOMER_DATA_ORIGIN_CODE in('ZAFO','ZSSG','ZMYN'))
and COUNTRY_ISO_CODE||CORPORATE_BRAND_SRC_NO not in ('AU1',
'DK3',
'BG3',
'CY3',
'EE3',
'FI3',
'GR3',
'LV3',
'LT3',
'SI3',
'SK3',
'LU3',
'HR3',
'HU3',
'CZ3',
'RO3',
'PL3')
and B.source_id='48'),
total_bp as (
SELECT DATE(MEMBER_CREATION_DATETIME) as MEMBER_CREATION_DATETIME,COUNT(DISTINCT business_partner_no) as total_business_partner_no
from bp
group by DATE(MEMBER_CREATION_DATETIME)
order by DATE(MEMBER_CREATION_DATETIME) desc),
guest_bp as (
  SELECT DATE(MEMBER_CREATION_DATETIME) as MEMBER_CREATION_DATETIME,COUNT(DISTINCT business_partner_no) as guest_business_partner_no
from bp
where Guest_Checkout ='Y'
group by DATE(MEMBER_CREATION_DATETIME)
order by DATE(MEMBER_CREATION_DATETIME) desc
),
registered_bp as (SELECT DATE(MEMBER_CREATION_DATETIME) as MEMBER_CREATION_DATETIME,COUNT(DISTINCT business_partner_no) as registered_business_partner_no
from bp
where Guest_Checkout ='N'
group by DATE(MEMBER_CREATION_DATETIME)
order by DATE(MEMBER_CREATION_DATETIME) desc)
SELECT t.MEMBER_CREATION_DATETIME,t.total_business_partner_no,g.guest_business_partner_no,r.registered_business_partner_no FROM total_bp t
left join guest_bp g on g.MEMBER_CREATION_DATETIME=t.MEMBER_CREATION_DATETIME
left join registered_bp r on r.MEMBER_CREATION_DATETIME=t.MEMBER_CREATION_DATETIME
