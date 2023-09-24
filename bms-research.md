# Functions of a BMS
A BMS is basically supposed to protect, against:
*   overcharge
*   overdischarge
*   overcurrent

For each cell.

# Observations
*   DW01A IC is used. protection IC for lithium cells. https://datasheet.lcsc.com/lcsc/2007091535_PJSEMI-DW01_C686633.pdf
*   BB3A IC. charge/cell balance + some protection features. https://datasheet.lcsc.com/szlcsc/HYCON-Tech-HY2213-BB3A_C113632.pdf
*   TL431. not efficient. using this increases more resistors, so more power loss

*   ITU 2022 uses a custom board
*   Yildiz 2022 uses rechargeable 18650 cell battery
    *   designed BMS
    *   using DW01A, BB3A
*   NOrtheastern 2022 - custom
*   zeus (steve) 2022 - custom
*   BYU 2022 - hybrid. off-the-shelf bms, but gives signals to main controllers and a screen
*   monash 2022 - probably custom
*   saddleback college 2022 - custom
*   QzU 2022 - custom, using CAN

*   Some terms used in BMS:
    *   SoH - state of health, the ratio of the current capacity to the original capacity. $SoH = \frac{Q}{Q_{original}}$
    *   SoC - state of charge, the "level" of the battery. $SoC = \frac{Q}{Q_{max}}$
    *   RVL - remaining valid life. this a more complex metric, often predicted by using AI/ML models or statistics

*   Schoktty diodes are often used. faster switching, more capacity
*   if we want a simple BMS, we may consider a solution called PCM (protection circuit modules)
*   PCM just provides overcharge, overdischarge, overcurrent protection. no balancing. this is typically built with just the two ICs (DW01A, BB3A), capacitors, and resistors.

*   can use comparators to compare voltages of cell. use a known reference (like 4.09 V, 2.5 V etc) and adjust it using voltage dividers. then use it to compare for over-voltage, and similarly for under-voltage.
*   can also use an IC like TL431 to provide an adjustable reference voltage. this is very stable.
*   shunt resistors have an efficiency problem. they dissipate a lot of power, even at a low voltage (because we have high current).
*   similar approach can be taken for overcurrent protection. use a shunt resistor, and use a comparator to compare the voltage drop across the resistor to a known reference. shut resistor should be parallel to battery and low resistance to not affect the circuit much.
*   mosfets can be chained in series to form AND gates, with one input for each cell undervoltage protection, and one for the overcurrent protection. the output of this can go to a relay, which can disconnect the battery from the load.
