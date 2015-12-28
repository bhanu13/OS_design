Things I modified from last night (11/14) version:
---
1. Get the stuff compiled.

2. Changed function type push_stack_iret() and flush_tlb() to inline.
	Don't want to mess up the stack with extra function calling convention like pushing ebp and esp that kind of stuff.

3. Changed interface of terminal_read and terminal_write. Now their parameters are the same as system call.
	I got warning about incomplete parameter types so I just make all interface the same.

4. We CLI just before pushing stuff onto stack for IRET, but no STI after that. The solution is mentioned in http://www.jamesmolloy.co.uk/tutorial_html/10.-User%20Mode.html.
	Look for section 10.1.2 of the webpage.


Bugs/Problems known so far:
---
1. The interrupt flag IF is disable after the first context switch (which is our first execute_func("shell") in kernel.c).
	I don't know why. I'm pretty sure the IF flag is enable right after context switch (IRET).
	For now I just manually sti() at terminal.c:42.

2. Cannot execute any file or command in the shell. It just continue to next line and prompt. You will see when you try the shell.

Bhanu
TODO:
---
1. Fix read_data_file, read_dir for reading the file directory.

Problems:
---
4. Grep broken
		check read_data_file

5. Vid_map is missing.

6. Magic terminal_printf in rtc_read function. Though it works.


"FIXED"
---
2. ls does not print filename with 32 characters.
1. cat "programs" & "verylongname.txt"
	Maybe something goes wrong when switching inodes.
		inode number and pos changes in different inode blocks.
  cat RTC wait next tick
3. Counter
		May be working


---
1. terminal_printf temporarily down.
2. history support maybe
3. autocomplete TAB maybe
4. CHECK RTC_driver!!! (black magic line)



BUGS _ BA
...

1. Didn't restore 	// Restoring next_p_id value
	next_p_id[curr_term]--; when restarting shell missing from halt_func
	Moved it out of file_desc.c
	so instead of restarting, we would start a new one every time

2. Didn't switch the active and new processes properly
	active_progs[curr_term] = curr_pcb_ptr;
	curr_pcb_ptr = active_progs[dest_term];
	// In schedule.c

3. Not saving the esp0 of the 1st shell if -
	Run another prog
	Then switch terminals, switch saves it in the switch_terminals func
	- FIX - Add line
		// Save the ESP0
	curr_pcb_ptr->ESP0 = (uint32_t) prog_kstack_bottom;
	Commented in file_desc - OPTIONAL

4. Two set of EBP and ESP's. One is for halting the program, the other one for switching.

5. Use of curr_pcb_ptr instead of pcb_prog when curr_pcb_ptr is not updated yet.

6. After general protection exception, we should clear the terminal buffer.
	Otherwise the buffer is filled by other stuff.
	* Consider adding critical sections when switching and clearing buffer.

7. Sometimes we cannot debug/print lines in the shell.
	* The priority of TERM_READ_FLAG is higher than TERM_WRITE_FLAG. If we try to print something during a terminal_read, then it will not show things since it's using terminal_read conditions.


---
8. We needed to change disp_term in switch_terminals instead of curr_term

9. terminal_scroll_down - missing functionality for the 2 cases.

---
Note:

1. Update if disp_term is requesting some terminal_R/W but user typing things when curr_term(different) is currently running.
2. Write to buffer when disp_term and curr_term is different. Otherwise write to both vid_men and buffer.

