add_llvm_component_library(LLVMAsmPrinter
  AccelTable.cpp
  AddressPool.cpp
  AIXException.cpp
  ARMException.cpp
  AsmPrinter.cpp
  AsmPrinterDwarf.cpp
  AsmPrinterInlineAsm.cpp
  DbgEntityHistoryCalculator.cpp
  DebugHandlerBase.cpp
  DebugLocStream.cpp
  DIE.cpp
  DIEHash.cpp
  DwarfCFIException.cpp
  DwarfCompileUnit.cpp
  DwarfDebug.cpp
  DwarfExpression.cpp
  DwarfFile.cpp
  DwarfStringPool.cpp
  DwarfUnit.cpp
  EHStreamer.cpp
  ErlangGCPrinter.cpp
  OcamlGCPrinter.cpp
  PseudoProbePrinter.cpp
  WinCFGuard.cpp
  WinException.cpp
  CodeViewDebug.cpp
  WasmException.cpp

  DEPENDS
  intrinsics_gen

  LINK_COMPONENTS
  Analysis
  BinaryFormat
  CodeGen
  CodeGenTypes
  Core
  DebugInfoCodeView
  DebugInfoDWARF
  MC
  MCParser
  Remarks
  Support
  Target
  TargetParser
  )

if (CMAKE_COMPILER_IS_GNUCXX)
set_source_files_properties(DwarfCompileUnit.cpp PROPERTIES
COMPILE_FLAGS -fno-strict-aliasing)
endif()

