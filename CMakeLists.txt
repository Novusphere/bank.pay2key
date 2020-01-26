cmake_minimum_required(VERSION 3.5)
project(pay2key VERSION 1.0.0)

find_package(eosio.cdt)

### Only generate the wasm
#add_executable( bank.pay2key.wasm bank.pay2key.cpp )

### Generate the wasm and abi
add_contract( bank.pay2key bank.pay2key bank.pay2key.cpp )
