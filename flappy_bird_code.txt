module flappy_2(output reg [7:0]position_R, position_B, position_G, 
output reg [2:0]S,output reg ring, output reg [0:2]level, input CLK, Clear, up, output reg A,B,C,D,E,F,G,output reg [0:3]b,output reg COM1,COM2);
	wire [7:0]p1;
	wire [2:0]S1;
	wire [7:0]p2;	
	wire [2:0]S2;
	wire [7:0]p3;	
	wire [2:0]S3;
	reg [1:0] dispaly_count;
	reg [2:0] ending_count; // 0~7
	reg [1:0] ringflag;
   reg COM;
	reg [0:3]a;
	reg [2:0] hp; //0~7
	reg [1:0] level_count; //2~0
	reg [1:0] level_change; //2~0
	reg [3:0]A_count,B_count;
	player_div C1(CLK, CLK_div);	//2500000
	bule_div C2(CLK, CLK_div2);	//2500000
	smallest_div C3(CLK, CLK_div3);//每count == 50000就會翻轉一次 最小的 應該就是一秒一秒看? 顯示時鐘 25,000,000
	green_div C4(CLK, CLK_div4); 	//6000000
	rand_blue_div C5(CLK, CLK_div5);//123465
	rand_green_div C6(CLK, CLK_div6);//654321
	divfreq11 C11(CLK, CLK_div11);// 秒數加一 25,000,000
	player M1(CLK_div, Clear, up, p1);	
   blue_wall F1(CLK_div2, CLK_div5,Clear, p2, S2,get1);
	green_Wall F2(CLK_div4, CLK_div5, Clear, p3, S3);
   initial 
	begin
	 ring = 0; //鳴叫
	 b = 4'b1111;//顯示血量
	 dispaly_count = 0; //來回顯示 玩家 綠牆 藍牆
	 ringflag = 0;
	 hp = 4;
	 level_count = 1;
	end
	
	always @(posedge CLK_div11, posedge Clear)//時鐘 關卡顯示
		begin 
			if(Clear) 	
				begin
					A_count <= 4'b0000;//秒
					B_count <= 4'b0000;//十位數的秒
					level_count = 1;
				end
			else if (A_count == 0 && B_count == 1)
				begin
					A_count<=4'b0000;
					B_count<=4'b0000;
					level_count = level_count + 1;
				end
			else if(hp>0) 
				begin
					if(A_count<4'b1001)//小於9 為個位數
						A_count <= A_count + 1'd1; //加十進位的1 
					else
						begin
						A_count<=4'b0000;
						B_count<=B_count+1'd1;
						end
				end	
		end
		
	always@(posedge CLK_div3)//顯示時鐘
		begin
			COM<=~COM;
			if(COM)
				begin
					a[0:3]<=A_count;//個位數 5 a[0:3] = 1010 
					COM1<=0;
					COM2<=1;
				end
			else
				begin
					a[0:3]<=B_count; //a[0:3]代表數字
					COM1<=1;//最左邊的
					COM2<=0;//左邊數來第二個
				end
				A = ~(a[0]&~a[1]&~a[2] | ~a[0]&a[2] | ~a[1]&~a[2]&~a[3] | ~a[0]&a[1]&a[3]);
				 B = ~(~a[0]&~a[1] | ~a[1]&~a[2] | ~a[0]&~a[2]&~a[3] | ~a[0]&a[2]&a[3]);// 
				 C = ~(~a[0]&a[1] | ~a[1]&~a[2] | ~a[0]&a[3]);
				 D = ~(a[0]&~a[1]&~a[2] | ~a[0]&~a[1]&a[2] | ~a[0]&a[2]&~a[3] | ~a[0]&a[1]&~a[2]&a[3] | ~a[1]&~a[2]&~a[3]);
				 E = ~(~a[1]&~a[2]&~a[3] | ~a[0]&a[2]&~a[3]);
				 F = ~(~a[0]&a[1]&~a[2] | ~a[0]&a[1]&~a[3] | a[0]&~a[1]&~a[2] | ~a[1]&~a[2]&~a[3]);
				 G = ~(a[0]&~a[1]&~a[2] | ~a[0]&~a[1]&a[2] | ~a[0]&a[1]&~a[2] | ~a[0]&a[2]&~a[3]);
		end

	always@(posedge ring,posedge Clear)//碰到就扣血
		begin
		   if(Clear )
				hp<=4; //命有4條
		   else if(hp >= 0)
				hp<=hp-1;				
		end
	always@(posedge CLK_div3,posedge Clear)//各種顯示
		begin
			integer i; //放begin上面不行? for
			if(Clear)
				begin
					position_R<=8'b11111110;//[7:0]玩家
					position_G<=8'b11111111;
					position_B<=8'b11111111;
					S<=0;	
				end
			else if ( level_count >= 3)
				begin
					ending_count = ending_count + 1; 
					if(ending_count>7)
						ending_count = 0; //0~7
					if (ending_count == 0)
						begin
							position_R<=8'b11110000;
							position_G<=8'b11110000;
							position_B<=8'b11110000;
							S<=0;	
						end
					else if (ending_count == 1)
						begin
							position_R<=8'b11111011;
							position_G<=8'b11111011;
							position_B<=8'b11111011;
							S<=1;	
						end
					else if (ending_count == 2)
						begin
							position_R<=8'b11111011;//
							position_G<=8'b11111011;
							position_B<=8'b11111011;
							S<=2;	
						end
					else if (ending_count == 3)
						begin
							position_R<=8'b11110000;
							position_G<=8'b11110000;
							position_B<=8'b11110000;
							S<=3;	
						end
					else if (ending_count == 4)
						begin
							position_R<=8'b11110001;
							position_G<=8'b11110001;
							position_B<=8'b11110001;
							S<=4;	
						end
					else if (ending_count == 5)
						begin
							position_R<=8'b11111010;
							position_G<=8'b11111010;
							position_B<=8'b11111010;
							S<=5;	
						end
					else if (ending_count == 6)
						begin
							position_R<=8'b11111010;
							position_G<=8'b11111010;
							position_B<=8'b11111010;
							S<=6;	
						end
					else if (ending_count == 7)
						begin
							position_R<=8'b11110001;
							position_G<=8'b11110001;
							position_B<=8'b11110001;
							S<=7;	
						end					
				end
			else if(hp > 0)
				begin
					dispaly_count = dispaly_count + 1; 
					if(dispaly_count>2)
						dispaly_count = 0;
					if(dispaly_count == 0)
						begin
							position_R<=~p1;
							position_G<=8'b11111111;
							position_B<=8'b11111111;
							S<=0;		
						end
					else if (dispaly_count == 1)
						begin
							position_R<=8'b11111111;
							position_G<=8'b11111111;
							position_B<=~p2;//row
							S<=S2;//col					
						end
					else if(level_count == 2 && dispaly_count == 2)
						begin
							position_R<=8'b11111111;
							position_G<=~p3;
							position_B<=8'b11111111;//row
							S<=S3;//col
						end
					///各種處裡///
					if(level_count <= 2)//第一關和第二關 都要看 藍色碰撞
						begin
						for(i = 0 ; i < 8; i = i + 1)begin//碰撞?
							if(( p1[i] == p2[i] &&(S1==S2) && p1[i] == 1))
								ringflag = 1;
							end	
						end
					if (level_count == 2)
						begin
						for(i = 0 ; i < 8; i = i + 1)begin//碰撞?
							if(( p1[i] == p3[i] &&(S1==S3) && p1[i] == 1))
								ringflag = 1;
							end
						end
					if (ringflag == 1)
						begin
							ring = 1;
							ringflag = 0;
						end
					else
						ring = 0;					
					for(i = 4; i > 0 ; i = i-1)begin//零血無法顯示
						if (hp >= i)//血量顯示
							b[i-1] = 1;//[0:3] hp = 3 b = 1110
						else
							b[i-1] = 0;
					end
					for(i = 3; i > 0 ; i = i-1)begin//零血無法顯示
						if (level_count >= i)//關卡顯示
							level[i-1] = 1;//[0:2]
						else
							level[i-1] = 0;
					end
					
				end		
			else //ending_count
				begin
					ending_count = ending_count + 1; 
					if(ending_count>7)
						ending_count = 0;
					if (ending_count == 0)
						begin
							position_R<=8'b11110000;
							position_G<=8'b11110000;
							position_B<=8'b11111111;
							S<=0;	
						end
					else if (ending_count == 1)
						begin
							position_R<=8'b11111101;
							position_G<=8'b11111101;
							position_B<=8'b11111111;
							S<=1;	
						end
					else if (ending_count == 2)
						begin
							position_R<=8'b11111011;//
							position_G<=8'b11111011;
							position_B<=8'b11111111;
							S<=2;	
						end
					else if (ending_count == 3)
						begin
							position_R<=8'b11110000;
							position_G<=8'b11110000;
							position_B<=8'b11111111;
							S<=3;	
						end
					else if (ending_count == 4)
						begin
							position_R<=8'b11111001;
							position_G<=8'b11111111;
							position_B<=8'b11111001;
							S<=4;	
						end
					else if (ending_count == 5)
						begin
							position_R<=8'b11110110;
							position_G<=8'b11111111;
							position_B<=8'b11110110;
							S<=5;	
						end
					else if (ending_count == 6)
						begin
							position_R<=8'b11110110;
							position_G<=8'b11111111;
							position_B<=8'b11110110;
							S<=6;	
						end
					else if (ending_count == 7)
						begin
							position_R<=8'b11111001;
							position_G<=8'b11111111;
							position_B<=8'b11111001;
							S<=7;	
						end					
				end	
		end
endmodule

//////移動物////////
module player(input CLK_div, Clear, up, output reg [7:0]position); 
	reg [2:0]row;
	always@(posedge CLK_div, posedge Clear)
		begin
			if(Clear)
				begin
					row <= 7;//最上面
					position<=8'b00000001;
				end			
			else if(up)
				begin
					if(position==8'b00000001)	//邊 row [7:0] 最上面是0
						row<=7;
					else
						row<=row+1;//最上面是0
				end			
			else
			begin
				if(position==8'b10000000)	//如果已經在最下面了 邊 最上面是0(最右)
					row<=0;
				else
					row<=row-1;
			end
			case(row)//最上面是0(最右)
				7:	position<= 8'b00000001;
				6:	position<= 8'b00000010;
				5:	position<= 8'b00000100;
				4:	position<= 8'b00001000;
				3:	position<= 8'b00010000;
				2:	position<= 8'b00100000;
				1:	position<= 8'b01000000;
				0:	position<= 8'b10000000;
			endcase
		end
endmodule

/////藍色障礙物///////
module blue_wall(input CLK_div2, CLK_div5,Clear,
					output reg [7:0]position_B, output reg [2:0]SS,output reg get);
	reg[24:0]cnt;
   reg restart;

	always @(posedge CLK_div5) 
		begin
			if(cnt > 250000)
				cnt <= 25'd0; 		
			else
		     cnt <= cnt + 1'b1;
		end
	always @(posedge CLK_div2)
		begin		
				if(Clear)
					begin
						SS <= 7;//col
						position_B<=8'b11111100;
					end
				if(SS == 0)
					begin
						SS <= 7;							
						case(cnt%8)//最上面是0(最右) 五選一
							3:	position_B<=8'b11111000; //
							4:	position_B<=8'b11110000;
							5:	position_B<=8'b11100000;
							6:	position_B<=8'b11000000;
							7:	position_B<=8'b10000000;//只有最下面一格
						endcase
					end
			   else
					SS <= SS - 1;		
		end

endmodule


/////綠色掉落物/////
module green_Wall(input CLK_div4, CLK_div5, Clear, 
				output reg [7:0]position, output reg [2:0]SS);
	reg[24:0]cnt;
   reg restart;

	always @(posedge CLK_div5) 
		begin
			if(cnt > 250000)
				cnt <= 25'd0; 		
			else
		     cnt <= cnt + 1'b1;
		end
	always @(posedge CLK_div4)
		begin		
				if(Clear)
					begin
						SS <= 7;
						position<=8'b00000011;
					end
				if(SS == 0)
					begin
						SS <= 7;							
						case(cnt%8)//最上面是0(最右) 五選一
							3:	position<=8'b00000001; //
							4:	position<=8'b00000011;
							5:	position<=8'b00000111;
							6:	position<=8'b00001111;
							7:	position<=8'b00001111;//只有最下面一格
						endcase
					end
			   else
					SS <= SS - 1;		
		end	
endmodule

//////移動物的除頻器//////
module player_div(input CLK, output reg CLK_div); 
	reg [24:0] Count; 
	always @(posedge CLK) 
		begin 
			if(Count > 2500000) //跟藍牆一樣
				begin 
					Count <= 25'b0; CLK_div <= ~CLK_div; 
				end 
			else 
				Count <= Count + 1'b1; 	
		end 
endmodule 

/////藍色掉落物的除頻器/////
module bule_div(input CLK, output reg CLK_div);
	reg [24:0] Count; 
	always @(posedge CLK) 
		begin 
			if(Count > 2500000) 
				begin 
					Count <= 25'b0; CLK_div <= ~CLK_div; 
				end 
			else 
				Count <= Count + 1'b1; 
		end 
endmodule 

//////移動物和掉落物交替的除頻器//////
module smallest_div(input CLK, output reg CLK_div); 
	reg [24:0] Count; 
	always @(posedge CLK) 
		begin 
			if(Count > 50000) 
				begin 
					Count <= 25'b0; CLK_div <= ~CLK_div; 
				end 
			else 
				Count <= Count + 1'b1; 
		end 
endmodule 

///////////綠色掉落物的除頻器//////////
module green_div(input CLK, output reg CLK_div); 
reg [24:0] Count; 
always @(posedge CLK) 
begin 
	if(Count > 6000000) 
	begin 
		Count <= 25'b0; CLK_div <= ~CLK_div; 
	end 
	else 
		Count <= Count + 1'b1; 
end 
endmodule 

////random////blue/////
module rand_blue_div(input CLK, output reg CLK_div); 
reg [24:0] Count; 
always @(posedge CLK) 
begin 
	if(Count > 123456) 
	begin 
		Count <= 25'b0; CLK_div <= ~CLK_div; 
	end 
	else 
		Count <= Count + 1'b1; 
end 
endmodule 

////random////green////
module rand_green_div(input CLK, output reg CLK_div); 
reg [24:0] Count; 
always @(posedge CLK) 
begin 
	if(Count > 654321) 
	begin 
		Count <= 25'b0; CLK_div <= ~CLK_div; 
	end 
	else 
		Count <= Count + 1'b1; 
end 
endmodule 
///加秒數///
module divfreq11(input CLK, output reg CLK_div);
  reg [29:0] Count;
  always @(posedge CLK)
    begin
    if(Count > 25000000)//因為一秒 50,000,000HZ 上下兩次才會有一次posedge
      begin
        Count <= 30'b0;//歸零
        CLK_div <= ~CLK_div;
      end
    else
      Count <= Count + 1'b1;
    end
endmodule