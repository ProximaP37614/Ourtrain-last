<script>
    import { onMount } from "svelte";
    import { redirect } from '@sveltejs/kit';
    import Swal from 'sweetalert2'
    
    let selectedSeatType = "";
    let selectedQuantity = 1;
    export let data;
    $: ({ loggedOnUserName, tripData, userInfo, availableSeats, travelTimeMinutes, old_seat_id} = data || {});
    
    // เพิ่มการคำนวณเวลาถึงปลายทาง
    $: arrivalTime = calculateArrivalTime(tripData?.fromDatetime, travelTimeMinutes);

    function calculateArrivalTime(fromDatetime, travelTimeMinutes) {
        if (!fromDatetime || !travelTimeMinutes) return '';
        const departureTime = new Date(fromDatetime);
        const arrivalTime = new Date(departureTime.getTime() + travelTimeMinutes * 60000);
        return arrivalTime.toLocaleTimeString('th-TH', {hour: '2-digit', minute:'2-digit'});
    }

    function translateSeatType(seatType) {
        const translations = {
            'seat1_room': 'ชั้น 1 ห้องส่วนตัว',
            'seat1_1downBed': 'ชั้น 1 เตียงล่าง',
            'seat1_1upBed': 'ชั้น 1 เตียงบน',
            'seat2_1upBed': 'ชั้น 2 เตียงบน (ปรับอากาศ)',
            'seat2_1downBed': 'ชั้น 2 เตียงล่าง (ปรับอากาศ)',
            'seat2_2upBed': 'ชั้น 2 เตียงบน (พัดลม)',
            'seat2_2downBed': 'ชั้น 2 เตียงล่าง (พัดลม)',
            'seat2_1normal': 'ชั้น 2 เบาะนั่ง (ปรับอากาศ)',
            'seat2_2normal': 'ชั้น 2 เบาะนั่ง (พัดลม)',
            'seat3_2normal': 'ชั้น 3'
        };
        return translations[seatType] || seatType;
    }

    // Calculate total price
    $: totalPrice = selectedSeatType
        ? availableSeats.find(seat => seat.seat_type === selectedSeatType)?.farePerSeat * selectedQuantity || 0
        : 0;

    function incrementQuantity() {
        if (selectedQuantity < 4) selectedQuantity++;
    }

    function decrementQuantity() {
        if (selectedQuantity > 1) selectedQuantity--;
    }

    async function confirmBooking() {
    console.log("เริ่มฟังก์ชัน confirmBooking");

    const bookingData = {
        userId: userInfo.passenger_id,
        tripId: tripData.tripId,
        seatType: selectedSeatType,
        quantity: selectedQuantity,
        totalPrice: totalPrice,
        fromStation: tripData.user_from_station,
        toStation: tripData.user_to_station,
        old_seat_id: tripData.old_seat_id
    };

    console.log("ข้อมูลการจองที่จะส่ง:", bookingData);

    try {
        const response = await fetch('reservation_change/api', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(bookingData),
        });

        const responseData = await response.json();
        console.log("สถานะการตอบกลับจากเซิร์ฟเวอร์:", response.status);

        if (response.ok) {
            console.log("การจองเสร็จสมบูรณ์ การตอบกลับจากเซิร์ฟเวอร์:", responseData);

            // เก็บข้อมูลใน sessionStorage
            sessionStorage.setItem('bookingData', JSON.stringify(bookingData));
            sessionStorage.setItem('paymentId', responseData.paymentId.toString());
            sessionStorage.setItem('operationType', 'change');
            console.log("ข้อมูลที่เก็บใน sessionStorage:", bookingData);
            console.log("บันทึกข้อมูลใน sessionStorage เรียบร้อย");

            alert(`การจองเสร็จสมบูรณ์!
                รหัสการชำระเงิน: ${responseData.paymentId}
                จำนวนเงิน: ${bookingData.totalPrice} บาท
                สถานะ: รอการชำระเงิน`);

            // นำผู้ใช้ไปยังหน้า payment
            setTimeout(() => {
                window.location.href = 'payment';
            }, 100);
        } else {
            console.error("ข้อผิดพลาดจากเซิร์ฟเวอร์:", responseData);
            alert('เกิดข้อผิดพลาดในการบันทึกการจอง: ' + responseData.message);
        }
    } catch (error) {
        console.error("เกิดข้อผิดพลาดใน confirmBooking:", error);
        alert('เกิดข้อผิดพลาดในการเชื่อมต่อกับเซิร์ฟเวอร์ กรุณาลองใหม่อีกครั้ง');
    }
}
if (response.ok) {
            console.log("การจองเสร็จสมบูรณ์ การตอบกลับจากเซิร์ฟเวอร์:", responseData);

            sessionStorage.setItem('paymentId', responseData.paymentId.toString());
            sessionStorage.setItem('totalPrice', bookingData.totalPrice.toString());

            console.log("บันทึกข้อมูลใน sessionStorage เรียบร้อย");

            // ใช้ SweetAlert เพื่อแสดงผลลัพธ์
            Swal.fire({
                icon: 'success',
                title: 'การจองเสร็จสมบูรณ์!',
                html: `
                    <p>รหัสการชำระเงิน: ${responseData.paymentId}</p>
                    <p>จำนวนเงิน: ${bookingData.totalPrice} บาท</p>
                    <p>สถานะ: รอการชำระเงิน</p>
                `,
                confirmButtonText: 'ไปยังหน้าชำระเงิน'
            }).then(() => {
                window.location.href = 'payment'; // ไปยังหน้าชำระเงินหลังจากผู้ใช้กดปุ่ม
            });
        } else {
            console.error("ข้อผิดพลาดจากเซิร์ฟเวอร์:", responseData);
            Swal.fire({
                icon: 'error',
                title: 'เกิดข้อผิดพลาด',
                text: 'เกิดข้อผิดพลาดในการจองตั๋วโปรดตรวจสอบข้อมูลการจองให้ครบถ้วน'
            });
        }
    } catch (error) {
        console.error("เกิดข้อผิดพลาดใน confirmBooking:", error);
        Swal.fire({
            icon: 'error',
            title: 'ข้อผิดพลาดในการเชื่อมต่อ',
            text: 'เกิดข้อผิดพลาดในการเชื่อมต่อกับเซิร์ฟเวอร์ กรุณาลองใหม่อีกครั้ง'
        });
    }
}
</script>

<div class="p-8">
    <main class="">
        <h2 class="text-3xl font-bold mb-4 text-[#102C57]">เปลี่ยนเเปลงตั๋วโดยสาร</h2>
        <p class="mb-6 text-sm text-gray-600">
            ท่านจำเป็นต้องกรอกข้อมูลต่อไปนี้ให้ครบถ้วน
            และดำเนินการชำระค่าธรรมเนียมตั๋วโดยสารภายใน 5
            นาทีหลังจากกดยืนยันการจองตั๋วโดยสาร การจองตั๋วโดยสารจึงสมบูรณ์
        </p>

        <div class="bg-blue-50 p-4 rounded-lg mb-6">
            <h3 class="font-semibold mb-2 text-blue-800">
                เที่ยวโดยสาร {tripData.tripName}
            </h3>
            <div class="flex space-x-36">
                <p class="text-[#102C57]">
                    <span class="font-semibold">ต้นทาง</span> {tripData.user_from_station}
                </p>
                <p class="text-[#102C57]">
                    <span class="font-semibold">ปลายทาง</span> {tripData.user_to_station}
                </p>
                <p class="text-[#102C57]">
                    <span class="font-semibold">วันที่</span>
                    {new Date(tripData.fromDatetime).toLocaleDateString('th-TH')}
                </p>
                <p class="text-[#102C57]">
                    <span class="font-semibold">เวลา</span>
                    {new Date(tripData.fromDatetime).toLocaleTimeString('th-TH', {hour: '2-digit', minute:'2-digit'})}น. - 
                    {new Date(tripData.toDatetime).toLocaleTimeString('th-TH', {hour: '2-digit', minute:'2-digit'})}น.
                </p>
            </div>
        </div>
        <div class="border-t border-gray-300 my-10"></div>
        <div class="grid grid-cols-2 items-center">
            <!-- Seat type selection -->
            <h3 class="font-semibold text-gray-800 mb-4">
                ชั้นโดยสาร-ประเภทที่นั่ง
            </h3>

            <!-- Number of seats -->
            <h3 class="font-semibold text-gray-800 mb-4">จำนวนที่นั่ง</h3>

            <!-- Dropdown for seat type selection -->
            <div class="relative w-3/4">
                <select
                    bind:value={selectedSeatType}
                    class="block appearance-none w-full bg-white border border-gray-300 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline"
                >
                    <option value="" disabled selected>เลือกประเภทที่นั่ง</option>
                    {#each availableSeats as seat}
                        <option value={seat.seat_type}>{translateSeatType(seat.seat_type)} (ว่าง: {seat.available_seats})</option>
                    {/each}
                </select>
                <div
                    class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700"
                >
                    <svg
                        class="fill-current h-4 w-4"
                        xmlns="http://www.w3.org/2000/svg"
                        viewBox="0 0 20 20"
                    ><path
                        d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"
                    /></svg>
                </div>
            </div>
    
            <!-- Buttons to increase/decrease number of seats -->
            <div class="flex items-center">
                <button
                    on:click={decrementQuantity}
                    class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded-l"
                >
                    -
                </button>
                <span class="bg-gray-100 py-2 px-4">{selectedQuantity}</span>
                <button
                    on:click={incrementQuantity}
                    class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded-r"
                >
                    +
                </button>
            </div>
        </div>
        <div class="border-t border-gray-300 my-12"></div>

        <div class="mt-8">
            <h3 class="font-semibold mb-4 text-[#102C57] text-lg">โปรดตรวจสอบข้อมูลต่อไปนี้</h3>
            <div class="grid grid-cols-4 gap-4">
                <p>ชื่อ: {userInfo.firstname || '-'}</p>
                <p>นามสกุล: {userInfo.lastname || '-'}</p>
                <p>เบอร์โทรศัพท์: {userInfo.phonenumber || '-'}</p>
                <p>อีเมล: {userInfo.email || '-'}</p>
                <p>เที่ยวโดยสาร: {tripData.tripName}</p>
                <p>จาก: {tripData.user_from_station} ถึง {tripData.user_to_station}</p>
                <p>วันที่เดินทาง: {new Date(tripData.fromDatetime).toLocaleDateString('th-TH')}</p>
                <p>เวลา: {new Date(tripData.fromDatetime).toLocaleTimeString('th-TH', {hour: '2-digit', minute:'2-digit'})}น. - 
                    {new Date(tripData.toDatetime).toLocaleTimeString('th-TH', {hour: '2-digit', minute:'2-digit'})}น.</p>
                <p>ชั้นโดยสาร: {selectedSeatType ? translateSeatType(selectedSeatType) : '-'}</p>
                <p>จำนวน: {selectedQuantity}</p>
                <p>ราคาต่อที่นั่ง: {selectedSeatType ? availableSeats.find(seat => seat.seat_type === selectedSeatType)?.farePerSeat.toFixed(2) : '-'} บาท</p>
                <p>ราคารวม: {totalPrice.toFixed(2)} บาท</p>
            </div>
        </div>

        <button
            style="background-color: #102C57;"
            class="mt-6 bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg transition duration-300 ease-in-out transform hover:-translate-y-1"
            on:click={confirmBooking}
        >
            ยืนยันการจองตั๋วโดยสาร
        </button>
    </main>
</div>