# The cxx.bnf grammar file
set(cxx_bnf ${CMAKE_CURRENT_SOURCE_DIR}/../lib/cxx.bnf)

# Using CMAKE_CROSSCOMPILING and not LLVM_USE_HOST_TOOLS because the latter is
# also set for LLVM_OPTIMIZED_TABLEGEN, which we don't care about here.
if(CMAKE_CROSSCOMPILING)
  build_native_tool(pseudo-gen pseudo_gen)
  set(pseudo_gen_target "${pseudo_gen}")
else()
  set(pseudo_gen $<TARGET_FILE:pseudo-gen>)
  set(pseudo_gen_target pseudo-gen)
endif()

# Generate inc files.
set(cxx_symbols_inc ${CMAKE_CURRENT_BINARY_DIR}/CXXSymbols.inc)
add_custom_command(OUTPUT ${cxx_symbols_inc}
   COMMAND "${pseudo_gen}"
     --grammar ${cxx_bnf}
     --emit-symbol-list
     -o ${cxx_symbols_inc}
   COMMENT "Generating nonterminal symbol file for cxx grammar..."
   DEPENDS ${pseudo_gen_target}
   VERBATIM)

set(cxx_bnf_inc ${CMAKE_CURRENT_BINARY_DIR}/CXXBNF.inc)
add_custom_command(OUTPUT ${cxx_bnf_inc}
   COMMAND "${pseudo_gen}"
     --grammar ${cxx_bnf}
     --emit-grammar-content
     -o ${cxx_bnf_inc}
   COMMENT "Generating bnf string file for cxx grammar..."
   DEPENDS ${pseudo_gen_target}
   VERBATIM)

# add_custom_command does not create a new target, we need to deine a target
# explicitly, so that other targets can depend on it.
add_custom_target(cxx_gen
    DEPENDS ${cxx_symbols_inc} ${cxx_bnf_inc}
    VERBATIM)
