import 'package:flutter/material.dart';
import 'package:syncfusion_flutter_calendar/calendar.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Advanced Calendar',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const AdvancedCalendarView(),
    );
  }
}

class AdvancedCalendarView extends StatefulWidget {
  const AdvancedCalendarView({super.key});
  @override
  State<AdvancedCalendarView> createState() => _AdvancedCalendarViewState();
}

class _AdvancedCalendarViewState extends State<AdvancedCalendarView> {
  final CalendarController _calendarController = CalendarController();
  List<Meeting> _appointments = [];

  @override
  void initState() {
    super.initState();
    _appointments = _getDataSource();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('高機能カレンダー')),
      body: SafeArea(
        child: SfCalendar(
          view: CalendarView.month,
          controller: _calendarController,
          dataSource: MeetingDataSource(_appointments),
          // 1. ドラッグ&ドロップとリサイズを有効化
          allowDragAndDrop: true,
          appointmentResizeMode: AppointmentResizeMode.appointment,
          
          // 2. カレンダーを画面下まで表示
          monthViewSettings: const MonthViewSettings(
            showAgenda: true,
            navigationDirection: MonthNavigationDirection.horizontal,
          ),
          
          // 3. 日付タップで1日表示に拡大アニメーション
          onTap: (CalendarTapDetails details) {
            if (_calendarController.view == CalendarView.month &&
                details.targetElement == CalendarElement.calendarCell) {
              _calendarController.view = CalendarView.day;
            } else if (_calendarController.view == CalendarView.day) {
              _calendarController.view = CalendarView.month;
            }
          },

          // 4. イベント移動・リサイズ時のコールバック
          onAppointmentResizeEnd: (AppointmentResizeEndDetails details) {
            final appointment = details.appointment as Meeting;
            setState(() {
              appointment.from = details.startTime ?? appointment.from;
              appointment.to = details.endTime ?? appointment.to;
            });
          },
          onDragEnd: (AppointmentDragEndDetails details) {
            final appointment = details.appointment as Meeting;
            setState(() {
              appointment.from = details.droppingTime!;
              // 期間を維持したまま移動
              appointment.to = details.droppingTime!.add(
                  appointment.to.difference(appointment.from));
            });
          },
          
          // 5. 不対応な色のハンドリング
          appointmentBuilder: (context, calendarAppointmentDetails) {
            final Meeting meeting = calendarAppointmentDetails.appointments.first;
            // 灰色対応：特定の明るい色以外はグレーにする等の処理
            final bool isSupportedColor = _isColorSupported(meeting.background);
            final color = isSupportedColor ? meeting.background : Colors.grey;

            return Container(
              color: color,
              child: Text(meeting.eventName, style: const TextStyle(color: Colors.white, fontSize: 10)),
            );
          },
        ),
      ),
    );
  }

  // サンプルデータ
  List<Meeting> _getDataSource() {
    final List<Meeting> meetings = <Meeting>[];
    final DateTime today = DateTime.now();
    meetings.add(Meeting(
      '打ち合わせ',
      today.add(const Duration(hours: 5, minutes: 0)),
      today.add(const Duration(hours: 6, minutes: 0)),
      const Color(0xFF0F8644), // サポート色
      false
    ));
    meetings.add(Meeting(
      '不対応カラーのイベント',
      today.add(const Duration(hours: 8, minutes: 0)),
      today.add(const Duration(hours: 10, minutes: 0)),
      Colors.black, // これがグレーになる
      false
    ));
    return meetings;
  }

  bool _isColorSupported(Color color) {
    // 灰色、黒、透明などを除外するロジック
    if (color == Colors.black || color == Colors.transparent || color == Colors.grey) {
      return false;
    }
    return true;
  }
}

// データモデル
class Meeting {
  Meeting(this.eventName, this.from, this.to, this.background, this.isAllDay);
  String eventName;
  DateTime from;
  DateTime to;
  Color background;
  bool isAllDay;
}

class MeetingDataSource extends CalendarDataSource {
  MeetingDataSource(List<Meeting> source) { appointments = source; }
  @override
  DateTime getStartTime(int index) => appointments![index].from;
  @override
  DateTime getEndTime(int index) => appointments![index].to;
  @override
  String getSubject(int index) => appointments![index].eventName;
  @override
  Color getColor(int index) => appointments![index].background;
  @override
  bool isAllDay(int index) => appointments![index].isAllDay;
}
