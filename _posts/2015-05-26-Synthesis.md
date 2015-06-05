---
layout: post
title:  "FPGAs?"
date:   2015-05-26 14:54:32
category: Hardware
---

### VHDL: ancientness aways!

{% highlight VHDL %}
library ieee;
use ieee.std_logic_1164.all;
use ieee.numeric_std.all;

ENTITY io_led IS
	PORT(	
		clock 		: in std_logic;
		reset 		: in std_logic;
		leds_status : in std_logic_vector(3 downto 0);
		leds_output : out std_logic_vector(3 downto 0)
	);
END io_led;

ARCHITECTURE behaviour OF io_led IS
	
BEGIN
	PROCESS(clock)
	BEGIN
		IF rising_edge(clock) THEN
			leds_output <= leds_status;
		END IF;
	END PROCESS;
END;

{% endhighlight %}
