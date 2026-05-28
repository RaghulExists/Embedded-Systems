

## Module-3
Introduction to ARM7 Instruction Set

3.3 Load-Store Instructions
•Load-storeinstructionstransferdatabetweenmemoryandprocessor
registers.
•Therearethreetypesofload-storeinstructions:single-register
transfer,multiple-registertransfer,andswap.
•3.3.1Single-RegisterTransfer
•Theseinstructionsareusedformovingasingledataiteminandout
ofaregister.
•Thedatatypessupportedaresignedandunsignedwords(32-bit),
halfwords(16-bit),andbytes.
•Herearethevariousload-storesingle-registertransferinstructions.





## Module 3 Questions
•Explain the Multiple Register load-store instructions with addressing
modes.
•Explain LDR instruction with pre-index and post-index using examples.
•Explain the STMFD stack instruction with an example.
•Explain Swap instruction of ARM7 with an example.
•Discuss the following index methods used in ARM with examples
i) Pre-indexing with write back upii) Pre-indexing   iii)Post indexing
•Describe LDMIB Instructions used in ARM with examples.
•Explain the STMED stack instruction with an example.

•Example 3.15
•LDRandSTRinstructionscanloadandstoredataonaboundaryalignmentthatisthesameasthedatatypesizebeing
loadedorstored.Forexample,LDRcanonlyload32-bitwordsonamemoryaddressthatisamultipleoffourbytes—0,
## 4,8,andsoon.
•Thisexampleshowsaloadfromamemoryaddresscontainedinregisterr1,followedbyastorebacktothesame
addressinmemory.
## •;
## •;loadregisterr0withthecontentsofthememoryaddresspointedtobyregisterr1.
## •;
•LDRr0,[r1];=LDRr0,[r1,#0]
## •;
## •;storethecontentsofregisterr0tothememoryaddresspointedtobyregisterr1.
## •;
•STRr0,[r1];=STRr0,[r1,#0]
•Thefirstinstructionloadsawordfromtheaddressstoredinregisterr1andplacesitintoregisterr0.Thesecond
instructiongoestheotherwaybystoringthecontentsofregisterr0totheaddresscontainedinregisterr1.Theoffset
fromregisterr1iszero.Registerr1iscalledthebaseaddressregister.

•3.3.2 Single-Register Load-Store Addressing Modes
•The ARM instruction set provides different modes for
addressing memory.
•These modes incorporate one of the indexing methods:
preindexwith writeback, preindex, and postindex(see Table
## 3.4).



•Example 3.16
•Preindexwith writeback calculates an address from a base register plus address offset and then updates that address base register with the
new address. In contrast, the preindexoffset is the same as the preindexwith writeback but does not update the address base register.
•Postindexonly updates the address base register after the address is used. The preindexmode is useful for accessing an element in a data
structure. The postindexand preindexwith writeback modes are useful for traversing an array.
PRE          r0 = 0x00000000
r1 = 0x00090000
mem32[0x00009000] = 0x01010101
mem32[0x00009004] = 0x02020202
Preindexingwith writeback:
LDR r0, [r1, #4]!
POST(1)          r0 = 0x02020202
r1 = 0x00009004
## Preindexing:
LDR r0, [r1, #4]
POST(2)            r0 = 0x02020202
r1 = 0x00009000
## Postindexing:
LDR r0, [r1], #4
POST(3)      r0 = 0x01010101
r1 = 0x00009004

•Table3.5showstheaddressingmodesavailableforloadand
storeofa32-bitwordoranunsignedbyte.
•Asignedoffsetorregisterisdenotedby“+/−”,identifyingthatit
iseitherapositiveornegativeoffsetfromthebaseaddress
registerRn.
•Thebaseaddressregisterisapointertoabyteinmemory,and
theoffsetspecifiesanumberofbytes.



•Immediatemeanstheaddressiscalculatedusingthebaseaddress
registeranda12-bitoffsetencodedintheinstruction.
•Registermeanstheaddressiscalculatedusingthebaseaddressregister
andaspecificregister’scontents.
•Scaledmeanstheaddressiscalculatedusingthebaseaddressregister
andabarrelshiftoperation.
•Table3.6providesanexampleofthedifferentvariationsoftheLDR
instruction.
•Table3.7showstheaddressingmodesavailableonloadandstore
instructionsusing16-bithalfwordorsignedbytedata.
•Table3.8showsthevariationsforSTRHinstructions.







3.3.3 Multiple-Register
## Transfer
•Load-storemultipleinstructionscantransfermultipleregisters
betweenmemoryandtheprocessorinasingleinstruction.
•ThetransferoccursfromabaseaddressregisterRnpointing
intomemory.
•Multiple-registertransferinstructionsaremoreefficientfrom
single-registertransfersformovingblocksofdataaround
memoryandsavingandrestoringcontextandstacks.

•Table3.9showsthedifferentaddressingmodesfortheload-storemultiple
instructions.
•HereNisthenumberofregistersinthelistofregisters.
•Anysubsetofthecurrentbankofregisterscanbetransferredtomemory
orfetchedfrommemory.
•ThebaseregisterRndeterminesthesourceordestinationaddressfora
loadstoremultipleinstruction.
•Thisregistercanbeoptionallyupdatedfollowingthetransfer.
•ThisoccurswhenregisterRnisfollowedbythe!character,similiartothe
single-registerload-storeusingpreindexwithwriteback.



•Example
## •3.17
•Inthisexample,registerr0isthebaseregisterRnandisfollowedby
## !,indicatingthattheregisterisupdatedaftertheinstructionis
executed.
•Youwillnoticewithintheloadmultipleinstructionthattheregisters
arenotindividuallylisted.Insteadthe“-”characterisusedtoidentify
arangeofregisters.Inthiscasetherangeisfromregisterr1tor3
inclusive.
•Eachregistercanalsobelisted,usingacommatoseparateeach
registerwithin“{”and“}”brackets.

•PRE mem32[0x80018] = 0x03
## •mem32[0x80014] = 0x02
## •mem32[0x80010] = 0x01
## •r0 = 0x00080010
## •r1 = 0x00000000
## •r2 = 0x00000000
## •r3 = 0x00000000
•LDMIA r0!, {r1-r3}
•POST r0 = 0x0008001c
## •r1 = 0x00000001
## •r2 = 0x00000002
## •r3 = 0x00000003
•Figure 3.3 shows a graphical representation.



•Thebaseregisterr0pointstomemoryaddress0x80010inthePREcondition.
## Memoryaddresses0x80010,0x80014,and0x80018containthevalues1,2,and
## 3respectively.
•Aftertheloadmultipleinstructionexecutesregistersr1,r2,andr3containthese
valuesasshowninFigure3.4.Thebaseregisterr0nowpointstomemory
address0x8001cafterthelastloadedword.
•NowreplacetheLDMIAinstructionwithaloadmultipleandincrementbefore
LDMIBinstructionandusethesamePREconditions.Thefirstwordpointedtoby
registerr0isignoredandregisterr1isloadedfromthenextmemorylocationas
showninFigure3.5.
•Afterexecution,registerr0nowpointstothelastloadedmemorylocation.Thisis
incontrastwiththeLDMIAexample,whichpointedtothenextmemorylocation.





•ThedecrementversionsDAandDBoftheload-storemultiple
instructionsdecrementthestartaddressandthenstoreto
ascendingmemorylocations.
•Thisisequivalenttodescendingmemorybutaccessingthe
registerlistinreverseorder.
•Withtheincrementanddecrementloadmultiples,youcan
accessarraysforwardsorbackwards.Theyalsoallowforstack
pushandpulloperations.



•Table3.10showsalistofload-storemultipleinstructionpairs.If
youuseastorewithbaseupdate,thenthepairedload
instructionofthesamenumberofregisterswillreloadthedata
andrestorethebaseaddresspointer.
•Thisisusefulwhenyouneedtotemporarilysaveagroupof
registersandrestorethemlater.

•Example 3.18
•This example shows an STM increment before instruction followed by an LDM decrement after instruction.
•PRE r0 = 0x00009000
## •r1 = 0x00000009
## •r2 = 0x00000008
## •r3 = 0x00000007
•STMIB r0!, {r1-r3}
•MOV r1, #1
•MOV r2, #2
•MOV r3, #3
•PRE(2) r0 = 0x0000900c
## •r1 = 0x00000001
## •r2 = 0x00000002
## •r3 = 0x00000003
•LDMDA r0!, {r1-r3}
•POST r0 = 0x00009000
## •r1 = 0x00000009
## •r2 = 0x00000008
## •r3 = 0x00000007
•The STMIB instruction stores the values 7, 8, 9 to memory. We then corrupt register r1 to r3.
•The LDMDA reloads the original values and restores the base pointer r0.

•TheBNEisthebranchinstructionBwithaconditionmnemonicNE
## (notequal).
•Ifthepreviouscompareinstructionsetstheconditionflagstonot
equal,thebranchinstructionisexecuted.
•Figure3.6showsthememorymapoftheblockmemorycopyand
howtheroutinemovesthroughmemory.
•Theoreticallythisloopcantransfer32bytes(8words)intwo
instructions.
•Thesenumbersassumeaperfectmemorysystemwithfastmemory.



## 3.3.3.1 Stack Operations
•TheARMarchitectureusestheload-storemultipleinstructionsto
carryoutstackoperations.
•Thepopoperation(removingdatafromastack)usesaloadmultiple
instruction;similarly,thepushoperation(placingdataontothestack)
usesastoremultipleinstruction.
•Whenusingastackyouhavetodecidewhetherthestackwillgrow
upordowninmemory.
•Astackiseitherascending(A)ordescending(D).Ascendingstacks
growtowardshighermemoryaddresses;incontrast,descending
stacksgrowtowardslowermemoryaddresses.

•Whenyouuseafullstack(F),thestackpointersppointstoanaddressthatisthe
lastusedorfulllocation(i.e.,sppointstothelastitemonthestack).
•Incontrast,ifyouuseanemptystack(E)thesppointstoanaddressthatisthe
firstunusedoremptylocation(i.e.,itpointsafterthelastitemonthestack).
•Thereareanumberofload-storemultipleaddressingmodealiasesavailableto
supportstackoperations(seeTable3.11).Nexttothepopcolumnistheactual
loadmultipleinstructionequivalent.
•Forexample,afullascendingstackwouldhavethenotationFAappendedtothe
loadmultipleinstruction—LDMFA.ThiswouldbetranslatedintoanLDMDA
instruction.



•Example 3.20
•TheSTMFDinstructionpushesregistersontothestack,updatingthesp.
•Figure3.7showsapushontoafulldescendingstack.Youcanseethat
whenthestackgrowsthestackpointerpointstothelastfullentryinthe
stack.
•PREr1=0x00000002
## •r4=0x00000003
## •sp=0x00080014
•STMFDsp!,{r1,r4}
•POSTr1=0x00000002
## •r4=0x00000003
## •sp=0x0008000c



•Example 3.21
•In contrast, Figure 3.8 shows a push operation on an empty stack using the
STMED instruction.
•The STMED instruction pushes the registers onto the stack but updates register
spto point to the next empty location.
•PRE r1 = 0x00000002
## •r4 = 0x00000003
## •sp= 0x00080010
•STMED sp!, {r1,r4}
•POST r1 = 0x00000002
## •r4 = 0x00000003
## •sp= 0x00080008



## 3.3.4 Swap Instruction
•Theswapinstructionisaspecialcaseofaload-store
instruction.Itswapsthecontentsofmemorywiththecontentsof
aregister.
•Thisinstructionisanatomicoperation—itreadsandwritesa
locationinthesamebusoperation,preventinganyother
instructionfromreadingorwritingtothatlocationuntilit
completes.

•Example 3.22
•The swap instruction loads a word from memory into register r0 and overwrites
the memory with register r1.
•PRE mem32[0x9000] = 0x12345678
## •r0 = 0x00000000
## •r1 = 0x11112222
## •r2 = 0x00009000
•SWP r0, r1, [r2]
•POST mem32[0x9000] = 0x11112222
## •r0 = 0x12345678
## •r1 = 0x11112222
## •r2 = 0x00009000
•This instruction is particularly useful when implementing semaphores and mutual
exclusion in an operating system.
•You can see from the syntax that this instruction can also have a byte size
qualifier B, so this instruction allows for both a word and a byte swap.

•Example 3.23
•This example shows a simple data guard that can be used to protect data from
being written by another task.
•The SWP instruction “holds the bus” until the transaction is complete.
spin
MOV r1, =semaphore
MOV r2, #1
SWP r3, r2, [r1] ; hold the bus until complete
CMP r3, #1
BEQ spin
•The address pointed to by the semaphore either contains the value 0 or 1.
•When the semaphore equals 1, The resource is used by other processes.
•When the semaphore equals 0, The resource is not used by other processes.

## Module 3 Questions
•Explain the Multiple Register load-store instructions with addressing
modes.
•Explain LDR instruction with pre-index and post-index using examples.
•Explain the STMFD stack instruction with an example.
•Explain Swap instruction of ARM7 with an example.
•Discuss the following index methods used in ARM with examples
i) Pre-indexing with write back upii) Pre-indexing   iii)Post indexing
•Describe LDMIB Instructions used in ARM with examples.
•Explain the STMED stack instruction with an example.

## 3.4 Software Interrupt
## Instruction
•Asoftwareinterruptinstruction(SWI)causesasoftware
interruptexception,whichprovidesamechanismfor
applicationstocalloperatingsystemroutines.

•When the processor executes an SWI instruction, it sets the
program counter pc to the offset 0x8 in the vector table.
•The instruction also forces the processor mode to SVC, which
allows an operating system routine to be called in a privileged
mode.
•Each SWI instruction has an associated SWI number, which is
used to represent a particular function call or feature.

•Example 3.24
•HerewehaveasimpleexampleofanSWIcallwithSWInumber
0x123456,usedbyARMtoolkitsasadebuggingSWI.TypicallytheSWI
instructionisexecutedinusermode.
•PRE cpsr= nzcVqift_USER
## •pc = 0x00008000
•lr= 0x003fffff; lr= r14
## •r0 = 0x12
•0x00008000 SWI 0x123456
•POST cpsr= nzcVqIft_SVC
•spsr= nzcVqift_USER
## •pc = 0x00000008
## •lr= 0x00008004
## •r0 = 0x12

•SinceSWIinstructionsareusedtocalloperatingsystemroutines,youneedsome
formofparameterpassing.
•Thisisachievedusingregisters.Inthisexample,registerr0isusedtopassthe
parameter0x12.
•Thereturnvaluesarealsopassedbackviaregisters.
•CodecalledtheSWIhandlerisrequiredtoprocesstheSWIcall.
•ThehandlerobtainstheSWInumberusingtheaddressoftheexecuted
instruction,whichiscalculatedfromthelinkregisterlr.
•TheSWInumberisdeterminedby
•SWI_Number=<SWIinstruction>ANDNOT(0xff000000)
•HeretheSWIinstructionistheactual32-bitSWIinstructionexecutedbythe
processor.

## 3.5 Program Status Register
## Instructions
•TheARMinstructionsetprovidestwoinstructionstodirectlycontrolaprogramstatusregister(psr).
•TheMRSinstructiontransfersthecontentsofeitherthecpsrorspsrintoaregister;inthereverse
direction,theMSRinstructiontransfersthecontentsofaregisterintothecpsrorspsr.
•Togethertheseinstructionsareusedtoreadandwritethecpsrandspsr.
•Inthesyntaxyoucanseealabelcalledfields.Thiscanbeanycombinationofcontrol(c),extension
## (x),status(s),andflags(f).
•Thesefieldsrelatetoparticularbyteregionsinapsr,asshowninFigure3.9.
•Syntax:MRS{<cond>}Rd,<cpsr|spsr>
•MSR{<cond>}<cpsr|spsr>_<fields>,Rm
•MSR{<cond>}<cpsr|spsr>_<fields>,#immediate



•Thecfieldcontrolstheinterruptmasks,Thumbstate,and
processormode.
•Example3.26showshowtoenableIRQinterruptsbyclearing
theImask.
•ThisoperationinvolvesusingboththeMRSandMSR
instructionstoreadfromandthenwritetothecpsr.

•Example 3.26
•The MSR first copies the cpsrinto register r1. The BIC instruction clears bit 7 of r1.
Register r1 is then copied back into the cpsr, which enables IRQ interrupts.
•You can see from this example that this code preserves all the other settings in the cpsr
and only modifies the I bit in the control field.
PRE cpsr= nzcvqIFt_SVC
MRS r1, cpsr
BIC r1, r1, #0x80 ; 0b10000000
MSR cpsr_c, r1
POST cpsr= nzcvqiFt_SVC
•This example is in SVC mode. In user mode you can read all cpsrbits, but you can only
update the condition flag field f.

## 3.5.1 Coprocessor Instructions
•Coprocessorinstructionsareusedtoextendtheinstructionset.A
coprocessorcaneitherprovideadditionalcomputationcapabilityorbe
usedtocontrolthememorysubsystemincludingcachesandmemory
management.
•Thecoprocessorinstructionsincludedataprocessing,registertransfer,
andmemorytransferinstructions.
•Wewillprovideonlyashortoverviewsincetheseinstructionsare
coprocessorspecific.Notethattheseinstructionsareonlyusedbycores
withacoprocessor.
•Syntax:CDP{<cond>}cp,opcode1,Cd,Cn{,opcode2}
•<MRC|MCR>{<cond>}cp,opcode1,Rd,Cn,Cm{,opcode2}
•<LDC|STC>{<cond>}cp,Cd,addressing

•Inthesyntaxofthecoprocessorinstructions,thecpfieldrepresentsthe
coprocessornumberbetweenp0andp15.Theopcodefieldsdescribethe
operationtotakeplaceonthecoprocessor.
•TheCn,Cm,andCdfieldsdescriberegisterswithinthecoprocessor.The
coprocessoroperationsandregistersdependonthespecificcoprocessor
youareusing.Coprocessor15(CP15)isreservedforsystemcontrol
purposes,suchasmemorymanagement,writebuffercontrol,cache
control,andidentificationregisters.

•Example 3.27
•ThisexampleshowsaCP15registerbeingcopiedintoa
general-purposeregister.
•;transferringthecontentsofCP15registerc0toregisterr10
•MRCp15,0,r10,c0,c0,0
•HereCP15register-0containstheprocessoridentification
number.Thisregisteriscopiedintothegeneral-purpose
registerr10.

## 3.5.2 Coprocessor15
## Instruction Syntax
•CP15configurestheprocessorcoreandhasasetofdedicatedregisterstostoreconfigurationinformation,asshownin
## Example3.27.
•Avaluewrittenintoaregistersetsaconfigurationattribute—forexample,switchingonthecache.CP15iscalledthe
systemcontrolcoprocessor.
•BothMRCandMCRinstructionsareusedtoreadandwritetoCP15,whereregisterRdisthecoredestinationregister,
Cnistheprimaryregister,Cmisthesecondaryregister,andopcode2isasecondaryregistermodifier.
•Youmayoccasionallyhearsecondaryregisterscalled“extendedregisters.”
•Asanexample,hereistheinstructiontomovethecontentsofCP15controlregisterc1intoregisterr1oftheprocessor
core:
•MRCp15,0,r1,c1,c0,0
•WeuseashorthandnotationforCP15referencethatmakesreferringtoconfigurationregisterseasiertofollow.The
referencenotationusesthefollowingformat:
•CP15:cX:cY:Z

•Thefirstterm,CP15,definesitascoprocessor15.Thesecondterm,
aftertheseparatingcolon,istheprimaryregister.
•TheprimaryregisterXcanhaveavaluebetween0and15.
•Thethirdtermisthesecondaryorextendedregister.Thesecondary
registerYcanhaveavaluebetween0and15.
•Thelastterm,opcode2,isaninstructionmodifierandcanhavea
valuebetween0and7.Someoperationsmayalsouseanonzero
valuewofopcode1.
•WewritetheseasCP15:w:cX:cY:Z.

## 3.6 Loading Constants
•You might have noticed that there is no ARM instruction to move a 32-bit constant
into a register. SinceARMinstructionsare 32 bits in size, they obviously cannot
specify a general 32-bit constant.
•To aid programming there are two pseudoinstructionsto move a 32-bit value into
a register.
•Syntax: LDR Rd, =constant
•ADR Rd, label
•The first pseudoinstructionwrites a 32-bit constant to a register using whatever
instructions are available. It defaults to a memory read if the constant cannot be
encoded using other instructions.
•The second pseudoinstructionwrites a relative address into a register, which will
be encoded using a pc-relative expression.

•Example 3.28
•This example shows an LDR instruction loading a 32-bit constant 0xff00ffff
into register r0.
•LDR r0, [pc, #constant_number-8-{PC}]
## •:
## •constant_number
•DCD 0xff00ffff
•This example involves a memory access to load the constant, which can
be expensive for time-critical routines.
•Example 3.29 shows an alternative method to load the same constant into
register r0 by using an MVN instruction.

## 3.8 Conditional Execution
•MostARMinstructionsareconditionallyexecuted—youcanspecifythatthe
instructiononlyexecutesiftheconditioncodeflagspassagivencondition
ortest.
•Byusingconditionalexecutioninstructionsyoucanincreaseperformance
andcodedensity.Theconditionfieldisatwo-lettermnemonicappendedto
theinstructionmnemonic.ThedefaultmnemonicisAL,oralwaysexecute.
•Conditionalexecutionreducesthenumberofbranches,whichalso
reducesthenumberofpipelineflushesandthusimprovestheperformance
oftheexecutedcode.
•Conditionalexecutiondependsupontwocomponents:theconditionfield
andconditionflags.Theconditionfieldislocatedintheinstruction,andthe
conditionflagsarelocatedinthecpsr.

•Example 3.34
•This example shows an ADD instruction with the EQ condition appended. This instruction will only
be executed when the zero flag in the cpsris set to 1.
•; r0 = r1 + r2 if zero flag is set
•ADDEQ r0, r1, r2
•Only comparison instructions and data processing instructions with the S suffix appended to the
mnemonic update the condition flags in the cpsr. ■
•Example 3.35
•To help illustrate the advantage of conditional execution, we will take the simple C code fragment
shown in this example and compare the assembler output using nonconditional and conditional
instructions.
## •while (a!=b)
## •{
•if (a>b) a -= b; else b -= a;
## •}
•Let register r1 represent a and register r2 represent b. The following code fragment shows the same
algorithm written in ARM assembler. This example only uses conditional execution on the branch
instructions

•This example only uses conditional execution on the branch instructions
gcdCMP r1, r2
BEQ complete
BLT lessthan
SUB r1, r1, r2
B gcd
lessthan
SUB r2, r2, r1
B gcd
complete
## •...
•Now compare the same code with full conditional execution. As you can see, this dramatically reduces the
number of instructions:
## •gcd
•CMP r1, r2
•SUBGT r1, r1, r2
•SUBLT r2, r2, r1
•BNE gcd

Introduction to the Thumb
## Instruction Set
## 4.1 Thumb Register Usage
•InThumbstate,youdonothavedirectaccesstoallregisters.Onlythelowregistersr0to
r7arefullyaccessible,asshowninTable4.2.Thehigherregistersr8tor12areonly
accessiblewithMOV,ADD,orCMPinstructions.
•CMPandallthedataprocessinginstructionsthatoperateonlowregistersupdatethe
conditionflagsinthecpsr.YoumayhavenoticedfromtheThumbinstructionsetlistand
fromtheThumbregisterusagetablethatthereisnodirectaccesstothecpsrorspsr.
•Inotherwords,therearenoMSR-andMRS-equivalentThumbinstructions.
•Toalterthecpsrorspsr,youmustswitchintoARMstatetouseMSRandMRS.Similarly,
therearenocoprocessorinstructionsinThumbstate.
•YouneedtobeinARMstatetoaccessthecoprocessorforconfiguringcacheand
memorymanagement.



4.2 ARM-Thumb Interworking
•ARM-ThumbinterworkingisthenamegiventothemethodoflinkingARMandThumb
codetogetherforbothassemblyandC/C++.Ithandlesthetransitionbetweenthetwo
states.Extracode,calledaveneer,issometimesneededtocarryoutthetransition.
•ATPCSdefinestheARMandThumbprocedurecallstandards.
•TocallaThumbroutinefromanARMroutine,thecorehastochangestate.Thisstate
changeisshownintheTbitofthecpsr.TheBXandBLXbranchinstructionscausea
switchbetweenARMandThumbstatewhilebranchingtoaroutine.
•TheBXlrinstructionreturnsfromaroutine,alsowithastateswitchifnecessary.
•TheBLXinstructionwasintroducedinARMv5T.OnARMv4Tcoresthelinkerusesa
veneertoswitchstateonasubroutinecall.Insteadofcallingtheroutinedirectly,thelinker
callstheveneer,whichswitchestoThumbstateusingtheBXinstruction.

•There are two versions of the BX or BLX instructions: an ARM
instruction and a Thumb equivalent.
•The ARM BX instruction enters Thumb state only if bit 0 of the
address in Rn is set to binary 1; otherwise it enters ARM state.
•The Thumb BX instruction does the same.
•Syntax: BX Rm
•BLX Rm | label

•Example 4.2
•Replacing the BX instruction with BLX simplifies the calling of a Thumb
routine since it sets the return address in the link register lr:
## •CODE32
•LDR r0, =thumbRoutine+1 ; enter Thumb state
•BLX r0 ; jump to Thumb code
•; continue here
## •CODE16
thumbRoutine
ADD r1, #1
BX r14 ; return to ARM code and state

## 4.3 Other Branch Instructions
•There are two variations of the standard branch instruction, or B. The
first is similar to the ARM version and is conditionally executed; the
branch range is limited to a signed 8-bit immediate, or −256 to +254
bytes.
•The second version removes the conditional part of the instruction
and expands the effective branch range to a signed 11-bit
immediate, or −2048 to +2046 bytes.
•The conditional branch instruction is the only conditionally executed
instruction in Thumb state.
•Syntax: B<cond> label
•B label
•BL label

•The BL instruction is not conditionally executed and has an
approximate range of+/−4 MB.
•This range is possible because BL (and BLX) instructions are
translated into a pair of 16-bit Thumb instructions.
•The first instruction in the pair holds the high part of the
branchoffset, and the second the low part.
•These instructions must be used as a pair. The code here
shows the various instructions used to return from a BL
subroutine call:

## 4.4 Data Processing
## Instructions
•The data processing instructions manipulate data within registers. They include move instructions, arithmetic
instructions, shifts, logical instructions, comparison instructions, and multiply instructions. The Thumb data processing
instructions are a subset of theARMdata processing instructions.
•Syntax:
•<ADC|ADD|AND|BIC|EOR|MOV|MUL|MVN|NEG|ORR|SBC|SUB> Rd, Rm
•<ADD|ASR|LSL|LSR|ROR|SUB> Rd, Rn #immediate
•<ADD|MOV|SUB> Rd,#immediate
•<ADD|SUB> Rd,Rn,Rm
•ADD Rd,pc,#immediate
•ADD Rd,sp,#immediate
•<ADD|SUB> sp, #immediate
•<ASR|LSL|LSR|ROR> Rd,Rs
•<CMN|CMP|TST> Rn,Rm
•CMP Rn,#immediate
•MOV Rd,Rn





•TheseinstructionsfollowthesamestyleastheequivalentARMinstructions.MostThumb
dataprocessinginstructionsoperateonlowregistersandupdatethecpsr.Theexceptions
are
•MOVRd,Rn
•ADDRd,Rm
•CMPRn,Rm
•ADDsp,#immediate
•SUBsp,#immediate
•ADDRd,sp,#immediate
•ADDRd,pc,#immediate
•whichcanoperateonthehigherregistersr8–r14andthepc.Theseinstructions,exceptfor
CMP,donotupdatetheconditionflagsinthecpsrwhenusingthehigherregisters.The
CMPinstruction,however,alwaysupdatesthecpsr.

•Example 4.3
•This example shows a simple Thumb ADD instruction. It takes two
low registers r1 and r2 and adds them together. The result is then
placed into register r0, overwriting the original contents. The cpsris
also updated.
PRE cpsr= nzcvIFT_SVC
r1 = 0x80000000
r2 = 0x10000000
ADD r0, r1, r2
POST r0 = 0x90000000
cpsr= NzcvIFT_SVC

•Example 4.4
•ThumbdeviatesfromtheARMstyleinthatthebarrelshift
operations(ASR,LSL,LSR,andROR)areseparate
instructions.Thisexampleshowsthelogicalleftshift(LSL)
instructiontomultiplyregisterr2by2.
PRE r2 = 0x00000002
r4 = 0x00000001
LSL r2, r4
POST r2 = 0x00000004
r4 = 0x00000001

4.5 Single-Register Load-Store
## Instructions
•The Thumb instruction set supports load and storing registers,
or LDR and STR. These instructions use two preindexed
addressing modes: offset by register and offset by immediate.
•Syntax: <LDR|STR>{<B|H>} Rd, [Rn,#immediate]
•LDR{<H|SB|SH>} Rd,[Rn,Rm]
•STR{<B|H>} Rd,[Rn,Rm]
•LDR Rd,[pc,#immediate]
•<LDR|STR> Rd,[sp,#immediate]



•YoucanseethedifferentaddressingmodesinTable4.3.
•TheoffsetbyregisterusesabaseregisterRnplustheregister
offsetRm.ThesecondusesthesamebaseregisterRnplusa
## 5-bitimmediateoravaluedependentonthedatasize.
•The5-bitoffsetencodedintheinstructionismultipliedbyone
forbyteaccesses,twofor16-bitaccesses,andfourfor32-bit
accesses.



•Example 4.5
•This example shows two Thumb instructions that use a preindexaddressing mode. Both use the
same pre-condition.
•PRE mem32[0x90000] = 0x00000001
## •mem32[0x90004] = 0x00000002
## •mem32[0x90008] = 0x00000003
## •r0 = 0x00000000
## •r1 = 0x00090000
## •r4 = 0x00000004
•LDR r0, [r1, r4] ; register
•POST r0 = 0x00000002
## •r1 = 0x00090000
## •r4 = 0x00000004
•LDR r0, [r1, #0x4] ; immediate
•POST r0 = 0x00000002
•Both instructions carry out the same operation. The only difference is the second LDR uses a fixed
offset, whereas the first one depends on the value in register r4.

4.6 Multiple-Register Load-
## Store Instructions
•The Thumb versions of the load-store multiple instructions are
reduced forms of the ARM load-store multiple instructions. They only
support the increment after (IA) addressing mode.
•Here N is the number of registers in the list of registers. You can see
that these instructions always update the base register Rn after
execution.
•The base register and list of registers are limited to the low registers
r0 to r7.

•Example 4.6
•This example saves registers r1 to r3 to memory addresses 0x9000 to 0x900c. It
also updates base register r4. Note that the update character ! is not an option,
unlike with the ARM
•instruction set.
•PRE r1 = 0x00000001
## •r2 = 0x00000002
## •r3 = 0x00000003
## •r4 = 0x9000
•STMIA r4!,{r1,r2,r3}
•POST mem32[0x9000] = 0x00000001
## •mem32[0x9004] = 0x00000002
## •mem32[0x9008] = 0x00000003
## •r4 = 0x900c

## 4.7 Stack Instructions
•The Thumb stack operations are different from the equivalent
ARM instructions because
•they use the more traditional POP and PUSH concept.
•Syntax: POP {low_register_list{, pc}}
PUSH {low_register_list{, lr}}

•Theinterestingpointtonoteisthatthereisnostackpointerinthe
instruction.Thisisbecausethestackpointerisfixedasregisterr13
inThumboperationsandspisautomaticallyupdated.
•Thelistofregistersislimitedtothelowregistersr0tor7.
•ThePUSHregisterlistalsocanincludethelinkregisterlr;similarly
thePOPregisterlistcanincludethepc.Thisprovidessupportfor
subroutineentryandexit,asshowninExample4.7.
•Thestackinstructionsonlysupportfulldescendingstackoperations.

•Example 4.7
•InthisexampleweusethePOPandPUSHinstructions.Thesubroutine
ThumbRoutineiscalledusingabranchwithlink(BL)instruction.
;Callsubroutine
BLThumbRoutine
## ;continue
ThumbRoutine
PUSH{r1,lr};entersubroutine
MOVr0,#2
POP{r1,pc};returnfromsubroutine.
•Thelinkregisterlrispushedontothestackwithregisterr1.Uponreturn,
registerr1ispoppedoffthestack,aswellasthereturnaddressbeing
loadedintothepc.Thisreturnsfromthesubroutine.

## 4.8 Software Interrupt
## Instruction
•SimilartotheARMequivalent,theThumbsoftwareinterrupt(SWI)
instructioncausesasoftwareinterruptexception.
•IfanyinterruptorexceptionflagisraisedinThumbstate,theprocessor
automaticallyrevertsbacktoARMstatetohandletheexception.
•Syntax:SWIimmediate
•TheThumbSWIinstructionhasthesameeffectandnearlythesame
syntaxastheARMequivalent.ItdiffersinthattheSWInumberislimitedto
therange0to255anditisnotconditionallyexecuted.

•Example 4.8
•ThisexampleshowstheexecutionofaThumbSWIinstruction.Notethat
theprocessorgoesfromThumbstatetoARMstateafterexecution.
•PREcpsr=nzcVqifT_USER
## •pc=0x00008000
## •lr=0x003fffff;lr=r14
## •r0=0x12
•0x00008000SWI0x45
•POSTcpsr=nzcVqIft_SVC
•spsr=nzcVqifT_USER
## •pc=0x00000008
## •lr=0x00008002
## •r0=0x12

## •THE END