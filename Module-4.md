

## Module-4
Exception and
## Interrupt
## Handling

## Introduction
•Attheheartofanembeddedsystemlietheexception
handlers.Theyareresponsibleforhandlingerrors,
interrupts,andothereventsgeneratedbytheexternal
system.
•Efficienthandlerscandramaticallyimprovesystem
performance.Theprocessofdeterminingagoodhandling
methodcanbecomplicated,challenging,andfun.
•Inthischapterwewillcoverthetheoryandpractice
ofhandlingexceptions,andspecificallythehandling
ofinterruptsontheARMprocessor.
•TheARMprocessorhassevenexceptionsthatcanhalt
thenormalsequentialexecutionofinstructions:Data
Abort,FastInterruptRequest,InterruptRequest,
PrefetchAbort,SoftwareInterrupt,Reset,and
UndefinedInstruction.

•Thischapterisdividedintothreemainsections:
■Exceptionhandling:Exceptionhandlingcoversthe
specificdetailsofhowtheARMprocessorhandles
exceptions.
■Interrupts:ARMdefinesaninterruptasaspecial
typeofexception.Thissectiondiscussestheuseof
interruptrequests,aswellasintroducingsomeof
thecommonterms,features,andmechanisms
surroundinginterrupthandling.
■Interrupthandlingschemes:Thefinalsection
providesasetofinterrupthandlingmethods.
## Includedwitheachmethodisanexample
implementation.

## 9.1 Exception Handling
•Anexceptionisanyconditionthatneedstohaltthenormal
sequentialexecutionofinstructions.
•ExamplesarewhentheARMcoreisreset,whenaninstructionfetch
ormemoryaccessfails,whenanundefinedinstructionis
encountered,whenasoftwareinterruptinstructionisexecuted,or
whenanexternalinterrupthasbeenraised.
•Exceptionhandlingisthemethodofprocessingtheseexceptions.
•Mostexceptionshaveanassociatedsoftwareexceptionhandler—a
softwareroutinethatexecuteswhenanexceptionoccurs.For
instance,aDataAbortexceptionwillhaveaDataAborthandler.
•Thehandlerfirstdeterminesthecauseoftheexceptionandthen
servicestheexception.Servicingtakesplaceeitherwithinthe
handlerorbybranchingtoaspecificserviceroutine.TheReset
exceptionisaspecialcasesinceitisusedtoinitializean
embeddedsystem.

•This section covers the following exception
handling topics:
•ARM processor mode and exceptions
•Vector table
•Exception priorities
•Link register offsets

9.1.1 ARM Processor Exceptions
and Modes
•Table9.1liststheARMprocessorexceptions.Eachexception
causesthecoretoenteraspecificmode.Inaddition,anyofthe
ARMprocessormodescanbeenteredmanuallybychangingthecpsr.
•Userandsystemmodearetheonlytwomodesthatarenotentered
byacorrespondingexception,inotherwords,toenterthesemodes
youmustmodifythecpsr.
•Whenanexceptioncausesamodechange,thecoreautomatically
## savesthecpsrtothespsroftheexceptionmode
## savesthepctothelroftheexceptionmode
## setsthecpsrtotheexceptionmode
## setspctotheaddressoftheexceptionhandler
•Figure9.1showsasimplifiedviewofexceptionsandassociated
modes.NotethatwhenanexceptionoccurstheARMprocessoralways
switchestoARMstate.





## 9.1.2 Vector Table
•Chapter2introducedthevectortable—atableofaddresses
thattheARMcorebranchestowhenanexceptionisraised.
## Theseaddressescommonlycontainbranchinstructionsofone
ofthefollowingforms:
■B<address>—Thisbranchinstructionprovidesabranch
relativefromthepc.
■LDRpc,[pc,#offset]—Thisloadregisterinstructionloads
thehandleraddressfrommemorytothepc.Theaddressisan
absolute32-bitvaluestoredclosetothevectortable.
## Loadingthisabsoluteliteralvalueresultsinaslightdelay
inbranchingtoaspecifichandlerduetotheextramemory
access.However,youcanbranchtoanyaddressinmemory.
■LDRpc,[pc,#-0xff0]—Thisloadregisterinstructionloads
aspecificinterruptserviceroutineaddressfromaddress
0xfffff030tothepc.Thisspecificinstructionisonlyused
whenavectorinterruptcontrollerispresent(VICPL190).

■MOVpc,#immediate—Thismoveinstruction
copiesanimmediatevalueintothepc.Itlets
youspanthefulladdressspacebutatlimited
alignment.Theaddressmustbean8-bit
immediaterotatedrightbyanevennumberof
bits.
•Table9.2showstheexception,mode,andvector
tableoffsetforeachexception.



•Example 9.1
•Figure9.2showsatypicalvectortable.The
UndefinedInstructionentryisabranch
instructiontojumptotheundefinedhandler.
## Theothervectorsuseanindirectaddressjump
withtheLDRloadtopcinstruction.
•NoticethattheFIQhandleralsousestheLDR
loadtopcinstructionanddoesnottake
advantageofthefactthatthehandlercanbe
placedattheFIQvectorentrylocation.



## 9.1.3 Exception Priorities
•Exceptionscanoccursimultaneously,sotheprocessorhastoadopt
aprioritymechanism.
•Table9.3showsthevariousexceptionsthatoccurontheARM
processorandtheirassociatedprioritylevel.
•Forinstance,theResetexceptionisthehighestpriorityand
occurswhenpowerisappliedtotheprocessor.Thus,whenareset
occurs,ittakesprecedenceoverallotherexceptions.
•Similarly,whenaDataAbortoccurs,ittakesprecedenceoverall
otherexceptionsapartfromaResetexception.Thelowestpriority
levelissharedbytwoexceptions,theSoftwareInterruptand
UndefinedInstructionexceptions.
•CertainexceptionsalsodisableinterruptsbysettingtheIorF
bitsinthecpsr,asshowninTable9.3.
•Eachexceptionisdealtwithaccordingtotheprioritylevelset
outinTable9.3.Thefollowingisasummaryoftheexceptionsand
howtheyshouldbehandled,startingwiththehighest.



•TheResetexceptionisthehighestpriorityexceptionandis
alwaystakenwheneveritissignaled.Theresethandler
initializesthesystem,includingsettingupmemoryand
caches.
•Externalinterruptsourcesshouldbeinitializedbefore
enablingIRQorFIQinterruptstoavoidthepossibilityof
spuriousinterruptsoccurringbeforetheappropriatehandler
hasbeensetup.
•Theresethandlermustalsosetupthestackpointersfor
allprocessormodes.Duringthefirstfewinstructionsof
thehandler,itisassumedthatnoexceptionsorinterrupts
willoccur.
•ThecodeshouldbedesignedtoavoidSWIs,undefined
instructions,andmemoryaccessesthatmayabort,thatis,
thehandleriscarefullyimplementedtoavoidfurther
triggeringofanexception.

•Data Abort exceptions occur when the memory controller
or MMU indicates that an invalid memory address has
been accessed (for example, if there is no physical
memory for an address) or when the current code
attempts to read or write to memory without the correct
access permissions.
•An FIQ exception can be raised within a Data Abort
handler since FIQ exceptions are not disabled. When the
FIQ is completely serviced, control is returned back to
the Data Abort handler.
•A Fast Interrupt Request (FIQ) exception occurs when an
external peripheral sets the FIQ pin to nFIQ. An FIQ
exception is the highest priority interrupt. The core
disables both IRQ and FIQ exceptions on entry into the
FIQ handler.

•Thus, no external source can interrupt the processor
unless the IRQ and/or FIQ exceptions are reenabled by
software. It is desirable that the FIQ handler (and
also the abort, SWI, and IRQ handlers) is carefully
designed to service the exception efficiently.
•An Interrupt Request (IRQ) exception occurs when an
external peripheral sets the IRQ pin to nIRQ. An IRQ
exception is the second-highest priority interrupt.
•The IRQ handler will be entered if neither an FIQ
exception nor Data Abort exception occurs. On entry to
the IRQ handler, the IRQ exceptions are disabled and
should remain disabled until the current interrupt
source has been cleared.

•A Prefetch Abort exception occurs when an attempt to fetch an
instruction results in a memory fault. This exception is raised
when the instruction is in the execute stage of the pipeline and
if none of the higher exceptions have been raised.
•On entry to the handler, IRQ exceptions will be disabled, but the
FIQ exceptions will remain unchanged. If FIQ is enabled and an FIQ
exception occurs, it can be taken while servicing the Prefetch
## Abort.
•A Software Interrupt (SWI) exception occurs when the SWI
instruction is executed and none of the other higher-priority
exceptions have been flagged. On entry to the handler, the cpsr
will be set to supervisor mode.
•If the system uses nested SWI calls, the link register r14 and
spsrmust be stored away before branching to the nested SWI to
avoid possible corruption of the link register and the spsr.

•AnUndefinedInstructionexceptionoccurswhenan
instructionnotintheARMorThumbinstructionsetreaches
theexecutestageofthepipelineandnoneoftheother
exceptionshavebeenflagged.
•TheARMprocessor“asks”thecoprocessorsiftheycanhandle
thisasacoprocessorinstruction.Sincecoprocessorsfollow
thepipeline,instructionidentificationcantakeplacein
theexecutestageofthecore.
•Ifnoneofthecoprocessorsclaimstheinstruction,an
UndefinedInstructionexceptionisraised.
•BoththeSWIinstructionandUndefinedInstructionhavethe
samelevelofpriority,sincetheycannotoccuratthesame
time(inotherwords,theinstructionbeingexecutedcannot
bothbeanSWIinstructionandanundefinedinstruction).

## 9.1.4 Link Register Offsets
•When an exception occurs, the link register is set to a
specific address based on the current pc.
•For instance, when an IRQ exception is raised, the link
register lrpoints to the last executed instruction
plus 8. Care has to be taken to make sure the exception
handler does not corrupt lrbecause lris used to
return from an exception handler.
•The IRQ exception is taken only after the current
instruction is executed, so the return address has to
point to the next instruction, or lr− 4. Table 9.4
provides a list of useful addresses for the different
exceptions.
•The next three examples show different methods of
returning from an IRQ or FIQ exception handler



•Example 9.2
•This example shows that a typical method of returning from an IRQ and FIQ handler is to
use a SUBS instruction:
## •handler
•<handler code>
## •...
•SUBS pc, r14, #4 ; pc=r14-4
•Because there is an S at the end of the SUB instruction and the pc is the destination
register, the cpsris automatically restored from the spsrregister.■
•Example 9.3
•This example shows another method that subtracts the offset from the link register r14
at the beginning of the handler.
## •handler
•SUB r14, r14, #4 ; r14-=4
## •...
•<handler code>
## •...
•MOVS pc, r14 ; return
•After servicing is complete, return to normal execution occurs by moving the link
register r14 into the pc and restoring cpsrfrom the spsr.

•Example 9.4
•The final example uses the interrupt stack to store the link
register. This method first subtracts an offset from the link
register and then stores it onto the interrupt stack.
## •handler
•SUB r14, r14, #4 ; r14-=4
•STMFD r13!,{r0-r3, r14} ; store context
## •...
•<handler code>
## •...
•LDMFD r13!,{r0-r3, pc}ˆ ; return
•To return to normal execution, the LDM instruction is used to load
the pc. The ˆ symbol in the instruction forces the cpsrto be
restored from the spsr.

## 9.2 Interrupts
•There are two types of interrupts available on the ARM
processor. The first type of interrupt causes an
exception raised by an external peripheral—namely, IRQ
and FIQ.
•The second type is a specific instruction that causes
an exception—the SWI instruction.
•Both types suspend the normal flow of a program.
•In this section we will focus mainly on IRQ and FIQ
interrupts. We will cover these topics:
■Assigning interrupts
■Interrupt latency
■IRQ and FIQ exceptions
■Basic interrupt stack design and implementation

## 9.2.1 Assigning Interrupts
•A system designer can decide which hardware peripheral can
produce which interrupt request. This decision can be
implemented in hardware or software (or both) and depends
upon the embedded system being used.
•An interrupt controller connects multiple external
interrupts to one of the two ARM interrupt requests.
Sophisticated controllers can be programmed to allow an
external interrupt source to cause either an IRQ or FIQ
exception.
•When it comes to assigning interrupts, system designers have
adopted a standard design practice:
•■Software Interrupts are normally reserved to call
privileged operating system routines.
•For example, an SWI instruction can be used to change a
program running in user mode to a privileged mode.

■Interrupt Requests are normally assigned for
general-purpose interrupts. For example, a periodic
timer interrupt to force a context switch tends to
be an IRQ exception. The IRQ exception has a lower
priority and higher interrupt latency.
■Fast Interrupt Requests are normally reserved for a
single interrupt source that requires a fast
response time—for example, direct memory access
specifically used to move blocks of memory.
•Thus, in an embedded operating system design, the
FIQ exception is used for a specific application,
leaving the IRQ exception for more general
operating system activities.

## 9.2.2 Interrupt Latency
•Interrupt-driven embedded systems have to fight a battle with interrupt
latency—the interval of time from an external interrupt request signal
being raised to the first fetch of an instruction of a specific interrupt
service routine (ISR).
•Interrupt latency depends on a combination of hardware and software.
System architects must balance the system design to handle multiple
simultaneous interrupt sources and minimize interrupt latency. If the
interrupts are not handled in a timely manner, then the system will
exhibit slow response times.
•Software handlers have two main methods to minimize interrupt latency. The
first method is to use a nested interrupt handler, which allows further
interrupts to occur even when currently servicing an existing interrupt
## (see Figure 9.3).
•This is achieved by reenabling the interrupts as soon as the interrupt
source has been serviced (so it won’t generate more interrupts) but before
the interrupt handling is complete. Once a nested interrupt has been
serviced, then control is relinquished to the original interrupt service
routine.
•Normal execution



•Thesecondmethodinvolvesprioritization.You
programtheinterruptcontrollertoignore
interruptsofthesameorlowerprioritythanthe
interruptyouarehandling,soonlyahigher-
prioritytaskcaninterruptyourhandler.Youthen
reenableinterrupts.
•Theprocessorspendstimeinthelower-priority
interruptsuntilahigher-priorityinterrupt
occurs.
•Thereforehigher-priorityinterruptshavealower
averageinterruptlatencythanthelower-priority
interrupts,whichreduceslatencybyspeedingup
thecompletiontimeonthecriticaltime-sensitive
interrupts.

9.2.3 IRQ and FIQ Exceptions
•IRQ and FIQ exceptions only occur when a specific interrupt mask is cleared
in the cpsr. The ARM processor will continue executing the current
instruction in the execution stage of the pipeline before handling the
interrupt—an important factor in designing a deterministic interrupt handler
since some instructions require more cycles to complete the execution stage.
•An IRQ or FIQ exception causes the processor hardware to go through a
standard procedure (provided the interrupts are not masked):
•1. The processor changes to a specific interrupt request mode, which reflects
the interrupt being raised.
•2. The previous mode’s cpsris saved into the spsrof the new interrupt
request mode.
•3. The pc is saved in the lrof the new interrupt request mode.
•4. Interrupt/s are disabled—either the IRQ or both IRQ and FIQ exceptions are
disabled in the cpsr. This immediately stops another interrupt request of the
same type being raised.
•5. The processor branches to a specific entry in the vector table.
•The procedure varies slightly depending upon the type of interrupt being
raised. We will illustrate both interrupts with an example. The first example
shows what happens when an IRQ exception is raised, and the second example
shows what happens when an FIQ exception is raised.

•Example 9.5
•Figure 9.4 shows what happens when an IRQ exception is raised when the
processor is in user mode. The processor starts in state 1. In this
example both the IRQ and FIQ exception bits in the cpsrare enabled.
•When an IRQ occurs the processor moves into state 2. This transition
automatically sets the IRQ bit to one, disabling any further IRQ
exceptions. The FIQ exception, however, remains enabled because FIQ has a
higher priority and therefore does not get disabled when a low-priority
IRQ exception is raised.
•The cpsrprocessor mode changes to IRQ mode. The user mode cpsris
automatically copied into spsr_irq.
•Register r14_irq is assigned the value of the pc when the interrupt was
raised. The pc is then set to the IRQ entry +0x18 in the vector table.
•In state 3 the software handler takes over and calls the appropriate
interrupt service routine to service the source of the interrupt. Upon
completion, the processor mode reverts back to the original user mode code
in state 1.



•Example 9.6
•Figure9.5showsanexampleofanFIQexception.The
processorgoesthroughasimilarprocedureaswiththe
IRQexception,butinsteadofjustmaskingfurtherIRQ
exceptionsfromoccurring,theprocessoralsomasksout
furtherFIQexceptions.
•Thismeansthatbothinterruptsaredisabledwhen
enteringthesoftwarehandlerinstate3.
•ChangingtoFIQmodemeansthereisnorequirementto
saveregistersr8tor12sincetheseregistersare
bankedinFIQmode.Theseregisterscanbeusedtohold
temporarydata,suchasbufferpointersorcounters.
ThismakesFIQidealforservicingasingle-source,
high-priority,low-latencyinterrupt.



9.2.3.1 Enabling and Disabling
FIQ and IRQ Exceptions
•TheARMprocessorcorehasasimpleproceduretomanuallyenable
anddisableinterruptsthatinvolvesmodifyingthecpsrwhenthe
processorisinaprivilegedmode.
•Table9.5showshowIRQandFIQinterruptsareenabled.The
procedureusesthreeARMinstructions.
•ThefirstinstructionMRScopiesthecontentsofthecpsrinto
registerr1.ThesecondinstructionclearstheIRQorFIQmask
bit.Thethirdinstructionthencopiestheupdatedcontentsin
registerr1backintothecpsr,enablingtheinterruptrequest.
## Thepostfix_cidentifiesthatthebitfieldbeingupdatedisthe
controlfieldbit[7:0]ofthecpsr.
•Table9.6showsasimilarproceduretodisableormaskan
interruptrequest.
•Itisimportanttounderstandthattheinterruptrequestiseither
enabledordisabledonlyoncetheMSRinstructionhascompleted
theexecutionstageofthepipeline.Interruptscanstillbe
raisedormaskedpriortotheMSRcompletingthisstage.





## 9.2.4 Basic Interrupt Stack
Design and Implementation
## Exceptionshandlersmakeextensiveuseofstacks,witheachmode
havingadedicatedregistercontainingthestackpointer.Thedesign
oftheexceptionstacksdependsuponthesefactors:
■Operatingsystemrequirements—Eachoperatingsystemhasitsown
requirementsforstackdesign.
■Targethardware—Thetargethardwareprovidesaphysicallimitto
thesizeandpositioningofthestackinmemory.
## Twodesigndecisionsneedtobemadeforthestacks:
■Thelocationdetermineswhereinthememorymapthestackbegins.
MostARM-basedsystemsaredesignedwithastackthatdescends
downwards,withthetopofthestackatahighmemoryaddress.
■Stacksizedependsuponthetypeofhandler,nestedornonnested.A
nestedinterrupthandlerrequiresmorememoryspacesincethestack
willgrowwiththenumberofnestedinterrupts.

•Agoodstackdesigntriestoavoidstackoverflow—wherethe
stackextendsbeyondtheallocatedmemory—becauseitcauses
instabilityinembeddedsystems.Therearesoftwaretechniques
thatidentifyoverflowandthatallowcorrectivemeasuresto
takeplacetorepairthestackbeforeirreparablememory
corruptionoccurs.Thetwomainmethodsare(1)tousememory
protectionand(2)tocallastackcheckfunctionatthestart
ofeachroutine.
•TheIRQmodestackhastobesetupbeforeinterruptsare
enabled—normallyintheinitializationcodeforthesystem.It
isimportantthatthemaximumsizeofthestackisknownina
simpleembeddedsystem,sincethestacksizeisreservedinthe
initialstagesofboot-upbythefirmware.
•Figure9.6showstwotypicalmemorylayoutsinalinearaddress
space.Thefirstlayout,A,showsatraditionalstacklayout
withtheinterruptstackstoredunderneaththecodesegment.
•Thesecondlayout,B,showstheinterruptstackatthetopof
thememoryabovetheuserstack.ThemainadvantageoflayoutB
overAisthatBdoesnotcorruptthevectortablewhenastack
overflowoccurs,andsothesystemhasachancetocorrect
itselfwhenanoverflowhasbeenidentified.



## 9.3 Interrupt Handling Schemes
•In this final section we will introduce a number of different interrupt handling
schemes, ranging from the simple nonnestedinterrupt handler to the more complex grouped
prioritized interrupt handler. Each scheme is presented as a reference with a general
description plus an example implementation.
•The schemes covered are the following:
•■A nonnestedinterrupt handler handles and services individual interrupts sequentially.
It is the simplest interrupt handler.
•■A nested interrupt handler handles multiple interrupts without a priority assignment.
•■A reentrant interrupt handler handles multiple interrupts that can be prioritized.
•■A prioritized simple interrupt handler handles prioritized interrupts.
•■A prioritized standard interrupt handler handles higher-priority interrupts in a
shorter time than lower-priority interrupts.
•■A prioritized direct interrupt handler handles higher-priority interrupts in a shorter
time and goes directly to a specific service routine.
•■A prioritized grouped interrupt handler is a mechanism for handling interrupts that
are grouped into different priority levels.
•■A VIC PL190 based interrupt service routine shows how the vector interrupt controller
(VIC) changes the design of an interrupt service routine

9.3.1 NonnestedInterrupt
## Handler
•Thesimplestinterrupthandlerisahandlerthatisnonnested:the
interruptsaredisableduntilcontrolisreturnedbacktotheinterrupted
taskorprocess.Becauseanonnestedinterrupthandlercanonlyservicea
singleinterruptatatime,handlersofthisformarenotsuitablefor
complexembeddedsystemsthatservicemultipleinterruptswithdiffering
prioritylevels.
•Figure9.8showsthevariousstagesthatoccurwhenaninterruptisraised
inasystemthathasimplementedasimplenonnestedinterrupthandler:
•1.Disableinterrupt/s—WhentheIRQexceptionisraised,theARMprocessor
willdisablefurtherIRQexceptionsfromoccurring.Theprocessormodeis
settotheappropriateinterruptrequestmode,andthepreviouscpsris
copiedintothenewlyavailablespsr_{interruptrequestmode}.The
processorwillthensetthepctopointtothecorrectentryinthevector
tableandexecutetheinstruction.Thisinstructionwillalterthepcto
pointtothespecificinterrupthandler.
•2.Savecontext—Onentrythehandlercodesavesasubsetofthecurrent
processormodenonbankedregisters.



•3.Interrupthandler—Thehandlerthenidentifies
theexternalinterruptsourceandexecutesthe
appropriateinterruptserviceroutine(ISR).
•4.Interruptserviceroutine—TheISRservicesthe
externalinterruptsourceandresetsthe
interrupt.
•5.Restorecontext—TheISRreturnsbacktothe
interrupthandler,whichrestoresthecontext.
•6.Enableinterrupts—Finally,toreturnfromthe
interrupthandler,thespsr_{interruptrequest
mode}isrestoredbackintothecpsr.Thepcis
thensettothenextinstructionafterthe
interruptwasraised.

## 9.3.2 Nested Interrupt Handler
•A nested interrupt handler allows for another interrupt to occur within
the currently called handler. This is achieved by reenabling the
interrupts before the handler has fully serviced the current inerrupt.
•For a real-time system this feature increases the complexity of the system
but also improves its performance. The additional complexity introduces
the possibility of subtle timing issues that can cause a system failure,
and these subtle problems can be extremely difficult to resolve.
•A nested interrupt method is designed carefully so as to avoid these types
of problems. This is achieved by protecting the context restoration from
interruption, so that the next interrupt will not fill the stack (cause
stack overflow) or corrupt any of the Registers.
•The first goal of any nested interrupt handler is to respond to interrupts
quickly so the handler neither waits for asynchronous exceptions, nor
forces them to wait for the handler.
•The second goal is that execution of regular synchronous code is not
delayed while servicing the various interrupts.
•The increase in complexity means that the designers have to balance
efficiency with safety, by using a defensive coding style that assumes
problems will occur. The handler has to check the stack and protect
against register corruption where possible.



•Figure9.9showsanestedinterrupthandler.Ascan
beenseenfromthediagram,thehandlerisquiteabit
morecomplicatedthanthesimplenonnestedinterrupt
handler.
•Thenestedinterrupthandlerentrycodeisidenticalto
thesimplenonnestedinterrupthandler,exceptthaton
exit,thehandlertestsaflagthatisupdatedbythe
## ISR.
•Theflagindicateswhetherfurtherprocessingis
required.Iffurtherprocessingisnotrequired,then
theinterruptserviceroutineiscompleteandthe
handlercanexit.Iffurtherprocessingisrequired,
thehandlermaytakeseveralactions:reenabling
interruptsand/orperformingacontextswitch.

•Reenabling interrupts involves switching out of IRQ mode to
either SVC or system mode.
•Interrupts cannot simply be reenabled when in IRQ mode
because this would lead to possible link register r14_irq
corruption, especially if an interrupt occurred after the
execution of a BL instruction.
•Performing a context switch involves flattening (emptying)
the IRQ stack because the handler does not perform a context
switch while there is data on the IRQ stack.
•All registers saved on the IRQ stack must be transferred to
the task’s stack, typically on the SVC stack.
•The remaining registers must then be saved on the task
stack. They are transferred to a reserved block of memory on
the stack called a stack frame.

9.3.3 ReentrantInterrupt
## Handler
•Areentrantinterrupthandlerisamethodofhandlingmultipleinterrupts
whereinterruptsarefilteredbypriority,whichisimportantifthereis
arequirementthatinterruptswithhigherpriorityhavealowerlatency.
## Thistypeoffilteringcannotbeachievedusingtheconventionalnested
interrupthandler.
•Thebasicdifferencebetweenareentrantinterrupthandlerandanested
interrupthandleristhattheinterruptsarereenabledearlyoninthe
reentrantinterrupthandler,whichcanreduceinterruptlatency.
•AllinterruptsinareentrantinterrupthandlermustbeservicedinSVC,
system,undefinedinstruction,orabortmodeontheARMprocessor.
•Ifinterruptsarereenabledinaninterruptmodeandtheinterruptroutine
performsaBLsubroutinecallinstruction,thesubroutinereturnaddress
willbesetintheregisterr14_irq.
•Thisaddresswouldbesubsequentlydestroyedbyaninterrupt,whichwould
overwritethereturnaddressintoregisterr14_irq.Toavoidthis,the
interruptroutineshouldswapintoSVCorsystemmode.TheBLinstruction
canthenuseregisterr14_svctostorethesubroutinereturnaddress.The
interruptsmustbedisabledatthesourcebysettingabitinthe
interruptcontrollerbeforereenablinginterruptsviathecpsr.



•Ifinterruptsarereenabledinthecpsrbefore
processingiscompleteandtheinterruptsourceisnot
disabled,aninterruptwillbeimmediatelyregenerated,
leadingtoaninfiniteinterruptsequenceorrace
condition.
•Mostinterruptcontrollershaveaninterruptmask
registerthatallowsyoutomaskoutoneormore
interrupts,buttheremaininginterruptsarestill
enabled.
•Theinterruptstackisunusedsinceinterruptsare
servicedinSVCmode(forexample,onthetask’s
stack).InsteadtheIRQstackregisterr13isusedto
pointtoa12-bytestructurethatwillbeusedtostore
someregisterstemporarilyoninterruptentry.

•Itisparamounttoprioritizeinterruptsina
reentrantinterrupthandler.Iftheinterrupts
arenotprioritized,thesystemlatency
degradestothatofanestedinterrupthandler
becauselower-priorityinterruptswillbeable
topreempttheservicingofahigher-priority
interrupt.
•Thisinturnleadstothelockingoutof
higher-priorityinterruptsforthedurationof
theservicingofalower-priorityinterrupt.

## 9.3.4 Prioritized Simple
## Interrupt Handler
•Both the nonnestedinterrupt handler and the
nested interrupt handler service interrupts on
a first-come-first-served basis. In comparison,
the prioritized interrupt handler will
associate a priority level with a particular
interrupt source.
•The priority level is used to dictate the order
that the interrupts will be serviced. Thus, a
higher-priority interrupt will take precedence
over a lower-priority interrupt, which is a
particularly desirable characteristic in many
embedded systems.

•Methods of handling prioritization can either be achieved in hardware or
software. For hardware prioritization, the handler is simpler to design
since the interrupt controller will provide the current highest-priority
interrupt that requires servicing.
•These systems require more initialization code at startup since the
interrupts and associated priority level tables have to be constructed
before the system can be switched on; software prioritization, on the
other hand, requires the additional assistance of an external interrupt
controller.
•This interrupt controller has to provide a minimal set of functions that
include being able to set and un-setmasks, and to read the interrupt
status and source.
•The rest of this section will cover a software prioritization technique
chosen because it is a general method and does not rely on a specialized
interrupt controller. To help describe the priority interrupt handler, we
will introduce a fictional interrupt controller based upon a standard
interrupt controller from ARM.
•The controller takes multiple interrupt sources and generates an IRQ
and/or FIQ signal depending upon whether a particular interrupt source is
enabled or disabled.
•Figure 9.11 shows a flow diagram of a simple priority interrupt handler,
based on a reentrant interrupt handler



## 9.3.5 Prioritized Standard
## Interrupt Handler
•Followingonfromtheprioritizedsimpleinterrupthandler,the
nexthandleraddsanadditionallevelofcomplexity.The
prioritizedsimpleinterrupthandlertestedalltheinterruptsto
establishthehighestpriority—aninefficientmethodof
establishingtheprioritylevelbutitdoeshavetheadvantageof
beingdeterministicsinceeachinterruptprioritywilltakethe
samelengthoftimetobeidentified.
•Analternativeapproachistojumpearlywhenthehighest-priority
interrupthasbeenidentified(seeFigure9.13),bysettingthepc
andjumpingimmediatelyoncetheprioritylevelhasbeen
established.Thismeansthattheidentificationsectionofthe
codefortheprioritizedstandardinterrupthandlerismore
involvedthanfortheprioritizedsimpleinterrupthandler.The
identificationsectionwilldeterminetheprioritylevelandjump
## •immediatelytoaroutinethatwillhandlethemaskingofthe
lower-priorityinterruptsandthenjumpagainviaajumptableto
theappropriateISR.



•The interrupt source can now be tested by comparing the highest to the lowest priority.
•The first priority level that matches the interrupt source determines the priority level
of the incoming interrupt because each interrupt has a preset priority level.
•Once a match is achieved, then the handler can branch to the routine that masks off the
lower-priority interrupts.
•To disable the equal-or lower-priority interrupts, the handler enters a routine that
first calculates the priority level using the base address in register r11 and link
register r14.
•Following the SUB instruction register r11 will now contain the value 4, 12, 20, or 28.
These values correspond to the priority level of the interrupt multiplied by eight plus
four.
•Register r11 is then divided by eight and added to the address of the priority_table.
Following the LDRB register r11 will equal one of the priority interrupt numbers (0, 1,
2, or 3).
•The priority mask can now be determined, using the technique of shifting left by two and
adding that to the register r10, which contains the address of the priority_mask.

•The base address for the interrupt controller is copied into register
r14_irq and is used to obtain the IRQ Enable register in the controller
and place it into register r12.
•Register r10 contains the new mask. The next step is to clear the lower-
priority interrupts using this mask by performing a binary AND with the
mask and r12 (IRQ Enable register) and storing the result into the IRQ
Enable Clear register. It is now safe to enable
•IRQ exceptions by clearing the ibit in the cpsr. Lastly the handler needs
to jump to the correct service routine, by modifying r11 (which still
contains the highest-priority interrupt) and the pc.
•Shifting register r11 left by two (multiplying r11 by four) and adding it
to the pc allows the handler to jump to the correct routine by loading the
address of the service routine directly into the pc. The jump table must
follow the instruction that loads the pc. There is an NOP between the jump
table and the LDR instruction that modifies the pc because the pc is
pointing two instructions ahead (or eight bytes).
•Note that the priority mask table is in interrupt bit order, and the
priority table is in priority order.

## 9.3.6 Prioritized Direct
## Interrupt Handler
•Onedifferencebetweentheprioritizeddirectinterrupthandler
andtheprioritizedstandardinterrupthandleristhatsomeofthe
processingismovedoutofthehandlerintotheindividualISRs.
•Themovedcodemasksoutthelower-priorityinterrupts.EachISR
willhavetomaskoutthelower-priorityinterruptsforthe
particularprioritylevel,whichcanbeafixednumbersincethe
prioritylevelhasalreadybeenpreviouslydetermined.
•Theseconddifferenceisthattheprioritizeddirectinterrupt
handlerjumpsdirectlytotheappropriateISR.EachISRis
responsiblefordisablingthelower-priorityinterruptsbefore
modifyingthecpsrtoreenableinterrupts.
•Thistypeofhandlerisrelativelysimplesincethemaskingis
donebytheindividualISR,butthereisasmallamountofcode
duplicationsinceeachinterruptserviceroutineiseffectively
carryingoutthesametask.

•The priority interrupt is established by checking the highest-priority
interrupt first and then working down to the lowest.
•Once a priority interrupt is identified, the pc is then loaded with the
address of the appropriate ISR. The indirect address is stored at the
address of the isr_tableplus the priority level shifted two bits to the
left (multiplied by four).
•Alternatively you could use a conditional branch BNE.
•The ISR jump table is r_tableis ordered with the highest-priority
interrupt at the beginning of the table.
•The service_timer1 entry shows an example of an ISR used in a priority
direct interrupt handler.
•Each ISR is unique and depends upon the particular interrupt source.
•A copy of the base address for the interrupt controller is placed into
register r14_irq.
•This address plus an offset is used to copy the IRQEnableregister into
register r12.

•The address of the priority mask table has to be copied into
register r10 so it can be used to calculate the address of the
actual mask.
•Register r11 is shifted left two positions, which gives an offset
of 0, 4, 8, or 12. The offset plus the address of the priority
mask table address is used to load the mask into register r10.
•Register r10 will contain the ISR mask, and register r12 will
contain the current mask.
•A binary AND is used to merge the two masks. Then the new mask is
used to configure the interrupt controller using the IRQ Enable
Clear register. It is now safe to enable IRQ exceptions by
clearing the ibit in the cpsr.
•The handler can continue servicing the current interrupt unless an
interrupt with a higher priority occurs, in which case that
interrupt will take precedence over the current interrupt

## 9.3.7 Prioritized Grouped
## Interrupt Handler
•Lastly,theprioritizedgroupedinterrupthandlerdiffers
fromtheotherprioritizedinterrupthandlerssinceitis
designedtohandlealargesetofinterrupts.
•Thisisachievedbygroupinginterruptstogetherandforming
asubset,whichcanthenbegivenaprioritylevel.
•Thedesignerofanembeddedsystemmustidentifyeachsubset
ofinterruptsourcesandassignagroupprioritylevelto
thatsubset.Itisimportanttobecarefulwhenselecting
thesubsetsofinterruptsourcessincethegroupscan
determinethecharacteristicsofthesystem.
•Groupingtheinterruptsourcestogethertendstoreducethe
complexityofthehandlersinceitisnotnecessarytoscan
througheveryinterrupttodeterminetheprioritylevel.
•Ifaprioritizedgroupedinterrupthandleriswelldesigned,
itwilldramaticallyimproveoverallsystemresponsetimes.

•The GROUP_xdefines assign the various interrupt sources to their
specific priority level by using a binary OR operation on the
binary patterns.
•The GMASK_xdefines assign the masks for the grouped interrupts.
The MASK_xdefines connect each GMASK_xto a specific interrupt
source, which can then be used in the priority mask table.
•After the context has been saved the interrupt handler loads the
IRQ status register using an offset from the interrupt controller
base address.
•The handler then identifies the group to which the interrupt
source belongs by using the binary AND operation on the source.
The letter S postfixed to the instructions means update condition
flags in the cpsr.
•Register r11 will now contain the highest-priority group 0 or 1.
The handler now masks out the other interrupt sources by applying
a binary AND operation with 0xf.

•The address of the lowest significant bit table is then loaded
into register r11. A byte isloadedfrom the start of the table
using the value in register r10 (0, 1, 2, or 3, see Table 9.12).
•Once the lowest significant bit position is loaded into register
r11, the handler branches to a routine.
•The disable_lower_priorityinterrupt routine first checks for a
spurious (no longer present) interrupt. If the interrupt is
spurious, then the unknown_conditionroutine is called.
•The handler then loads the IRQEnableregister and places the
result in register r12.
•The priority mask is found by loading in the address of the
priority mask table and then shifting the data in register r11
left by two. The result, 0, 4, 8, or 12, is added to the priority
mask address.
•Register r10 then contains a mask to disable the lower-priority
group interrupts from being raised.
•The next step is to clear the lower-priority interrupts using the
mask by performing a binary AND with the mask in registers r10 and
r12 (IRQEnableregister) and then clearing the bits by saving the
result into the IRQEnableClearregister. At this point it is now
safe to enable IRQ exceptions by clearing the ibit in the cpsr.

•Lastly the handler jumps to the correct
interrupt service routine by modifying register
r11 (which still contains the highest-priority
interrupt) and the pc.
•By shifting register r11 left by two and adding
the result to the pc the address of the ISR is
determined.
•This address is then loaded directly into the
pc. Note that the jump table must follow the
LDR instruction.
•The NOP is present due to the ARM pipeline.

## •
## THE END