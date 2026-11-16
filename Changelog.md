#  Changelog

#### Unreleased
- Fixed a lost wakeup in the synchronous card event wait (sleep/wake path): the completion now publishes the result and wakes the waiter under the command gate. Before, a completion that ran between the waiter's check and its sleep was lost and the waiter returned only at its deadline (10 s longer sleep, 30 s stall on wake).
- A card event completion that is dropped (the opposite event arrived first) or replaced (a second event of the same kind) is completed with `kIOReturnAborted` instead of being discarded, so a synchronous waiter returns at once and its context is freed.
- Fixed a kernel panic when a block request is serviced before the card is attached (e.g. right after a wake or at shutdown): requests wait for the pending insertion event, and fail with `kIOReturnNoMedia` if no card is attached.
- Pending block requests that are dropped (card removed) are now completed with an error; v0.9.7 returned them to the pool without a completion, so the caller waited forever. A request submitted for a removed card is never serviced on the next card.
- Wait 250 ms after the controller is re-initialized on wake before reading the card presence (boot argument `rtsxwakesettle`).
- Based on the report and patch in 0xFireWolf/RealtekCardReader#62.
- Fixed a kernel panic "Sleep transition timed out" caused by a lost card event completion (the waiter slept forever); the wait is now bounded and the completion is one-shot.
- Fixed the PCIe capability offsets (Link Control, Device Control 2, L1 substates were read from and written to non-existent addresses).
- The card reader returns to the idle state (ASPM, LTR, L1 substates) 200 ms after the last request, so the CPU package can reach C6 again. Boot argument `-rtsxnoidle` restores the old behavior.
- Fixed a kernel panic when a second insertion event arrives while a card is attached.
- Fixed the fallback from UHS-I to the high speed mode (the bus stays powered off for 100 ms, boot argument `iosdpoweroff`).
- Card insertions are debounced for 200 ms (boot argument `rtsxdebounce`); a pending insertion is dropped before the system sleeps.
- The power-management diagnostics (`RTSX Power` property) are only published with the boot argument `-rtsxdiag` or in debug builds.
- Removed build host paths from the binary.

#### v0.9.7 Beta
- ~~Fixed an issue that the card is not mounted after the computer wakes up when sleeps with the card inserted.~~
- Fixed a race condition that may resume the polling thread even though the client requests to pause it.
- Fixed an issue that the driver cannot be loaded on macOS Mojave. (Thanks @reelgirly)
- Cards that have failed to initialize at its maximum speed mode will be initialized at a lower speed mode.

#### v0.9.6 Beta
- Added support for RTS5260.
- Fixed an issue that the driver is not loaded if the USB card reader is not configured and thus does not expose its host interface.
- Addressed an issue that the driver fails to find the bulk endpoints of the USB card reader.
- Resolved an issue that the driver fails to initialize the USB card reader properly.
- Resolved an issue that the driver cannot detect whether the user has inserted/removed the card to/from the USB card reader.
- Added an option to delay the initialization of the card found during the system boots.
- Corrected the address of the USB-based card reader register that controls the SSC depth.
- Fixed an issue that System Information does not list the USB-based card reader properly.
- Resolved an issue that commands are timed out after the driver switches the bus clock of the card reader.
- Resolved an issue that USB-based card readers fail to initialize UHS-I cards properly.
- Pause the polling thread of the USB-based card reader when the host driver is processing a card event.
- Skip setting the bus power mode if the card reader is already operating at the requested mode.
- Resolved an issue that the card clock is incorrectly toggled thus fails to switch to 1.8V after the host device sends the CMD11.
- Revise the design of the host driver stack to make all data structures that are related to host requests device-independent.
- Optimize the host driver stack to avoid unnecessary bounce buffer allocations and data copies in card reader controllers.
- Minimize the amount of time the transfer buffer is ready for DMA to reduce the pressure in the low 4GB of physical memory.
- Addressed an issue that CMD17 and CMD24 fail to transfer any data from/to the card with USB-based card readers.
- Switched the mode from `AutoRead2` to `AutoRead3` when the host device services an inbound DMA transfer request.
- Resolved an issue that bulk pipes halted after the driver issued a DMA transfer request to USB-based card readers. 
- Revise the implementation of processing block requests to fix the regression caused by the new driver stack.
- Fixed an issue that the maximum segment size for DMA transfers is incorrectly defined for PCIe-based card readers.
- Fixed a race condition introduced by the polling thread that corrupts the response sent by USB-based card readers.
- Addressed an issue that UHS-I cards running in SDR50/104 mode are not tuned thus send invalid responses to USB-based card readers.
- Parse the SD status register value so that System Information can show the speed class of the card.
- System Information now shows the information of cards inserted into USB-based card readers.

#### v0.9.5 Beta
- Rebranded the project.
- Refactored the controller and the host device code to support USB-based card readers.
- Added support for RTS5129, RTS5139 and RTS5179.
- Reenabled the bonus feature but now requires the companion Lilu plugin [RealtekCardReaderFriend](https://github.com/0xFireWolf/RealtekCardReaderFriend/).

#### v0.9.4 Beta
- Fixed an issue that driving tables are not set for RTS5227 and RTS522A.  
- Disabled the bonus feature temporarily: Your card reader will not be listed in System Information.  
- Implemented the basic power management.
- Support macOS Mojave.

#### v0.9.3 Beta
- Added support for RTS5227 and RTS522A.

#### v0.9.2 Beta
- Fixed an issue that the block size and the total number of blocks are not reported properly for SDSC cards.
- Fixed an issue that a kernel panic occurs when the card is removed from the system without being ejected properly.
- Fixed an issue that the request event source can be reenabled after the card is removed from the system.
- Power off the bus when the card is removed from the system.
- Added support for RTS8402, RTS8411b and RTS8411.

#### v0.9.1 Beta
- Fixed an issue that `Info.plist` is malformed and thus the kext cannot be injected.
- Fixed an issue that the MSI index could not be probed correctly.
- Added support for RTS5209 and RTS5229.

#### v0.9.0 Beta
- Initial version for public beta testing.
