# Anonymisation in LIS

Source Lis Replica
select only inactive customers
check latest transaction date based on that do the anonymisation - checking the X years to pass per country

anonymise customer but leave archive - depends on configuration
time span to anonymise - the X years etc.
time span to delete archive - could be different then time span to anonymise
archive lis invoice

setting how often it should be executed - for instance one a month
only full ledger years for some countries

new anon request to remove data from archive

olli mattukinejn check balance
client type in client table
only clientType: payment - 1

when archive invoices has to be scheduled
there is new operation that creates another anonymisation requests with future start date to be executed someday
starter will start such request only when starting date will be less then current date