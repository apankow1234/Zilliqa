link_directories(${CMAKE_BINARY_DIR}/lib)


add_executable(zilliqa main.cpp)
add_custom_command(TARGET zilliqa
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:zilliqa> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(zilliqa PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(zilliqa PUBLIC Consensus Network Utils Zilliqa DirectoryService Node Validator NAT Boost::program_options)


add_executable(sendcmd sendcmd.cpp)
add_custom_command(TARGET zilliqa
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:sendcmd> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(sendcmd PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(sendcmd PUBLIC Network Utils Boost::program_options)

add_executable(genaccounts genaccounts.cpp)
add_custom_command(TARGET zilliqa
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:genaccounts> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(genaccounts PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(genaccounts PUBLIC AccountData ${Schnorr_LIBRARY} Utils Boost::program_options)

add_executable(genkeypair genkeypair.cpp)
add_custom_command(TARGET zilliqa
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:genkeypair> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(genkeypair PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(genkeypair PUBLIC ${Schnorr_LIBRARY} Utils -s)

add_executable(gentxn gentxn.cpp)
add_custom_command(TARGET zilliqa
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:gentxn> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(gentxn PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(gentxn PUBLIC AccountData Message Network Boost::program_options)

add_executable(signmultisig signmultisig.cpp)
add_custom_command(TARGET zilliqa
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:signmultisig> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(signmultisig PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(signmultisig PUBLIC ${Schnorr_LIBRARY} Utils Boost::program_options)

add_executable(verifymultisig verifymultisig.cpp)
add_custom_command(TARGET zilliqa
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:verifymultisig> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(verifymultisig PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(verifymultisig PUBLIC ${Schnorr_LIBRARY} Utils Boost::program_options)

add_executable(getaddr GetAddressFromPubKey.cpp)
add_custom_command(TARGET zilliqa
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:getaddr> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(getaddr PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(getaddr PUBLIC ${Schnorr_LIBRARY} Utils OpenSSL::Crypto Boost::program_options)

add_executable(getpub GetPubKeyFromPrivKey.cpp)
add_custom_command(TARGET zilliqa
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:getpub> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(getpub PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(getpub PUBLIC ${Schnorr_LIBRARY} Utils OpenSSL::Crypto Boost::program_options)

add_executable(validateDB validateDB.cpp)
add_custom_command(TARGET zilliqa
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:validateDB> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(validateDB PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(validateDB PUBLIC Node Mediator Validator -s)

add_executable(genTxnBodiesFromS3 genTxnBodiesFromS3.cpp)
add_custom_command(TARGET zilliqa
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:genTxnBodiesFromS3> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(genTxnBodiesFromS3 PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(genTxnBodiesFromS3 PUBLIC AccountStore AccountData Utils Persistence Server Boost::program_options)

add_executable(isolatedServer isolated_server.cpp)
add_custom_command(TARGET zilliqa
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:isolatedServer> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(isolatedServer PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(isolatedServer PUBLIC AccountStore AccountData Mediator Persistence Server Validator Boost::program_options)

add_executable(websocketsubscriber websocketsubscriber.cpp)
add_custom_command(TARGET zilliqa
        POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:websocketsubscriber> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(websocketsubscriber PUBLIC ${CMAKE_SOURCE_DIR}/src)
target_link_libraries(websocketsubscriber PUBLIC Utils Server AccountStore Boost::program_options)

add_executable(buildTxBlockHashesToNums buildTxBlockHashesToNums.cpp)
add_custom_command(TARGET zilliqa
       POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:buildTxBlockHashesToNums> ${CMAKE_BINARY_DIR}/tests/Zilliqa)
target_include_directories(buildTxBlockHashesToNums PUBLIC ${CMAKE_SOURCE_DIR}/src)

if (${CMAKE_CXX_COMPILER_ID} STREQUAL "AppleClang")
  target_link_libraries(buildTxBlockHashesToNums PUBLIC AccountData Persistence)
else()
  # FIXME: "-Wl,--start-group" was apparently added as a hack for some circular dependency; this shouldn't
  #        really be required and resolved by fixing the (circular) dependencies.
  target_link_libraries(buildTxBlockHashesToNums PUBLIC "-Wl,--start-group" AccountData Persistence)
endif()
