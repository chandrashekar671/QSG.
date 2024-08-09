mport 'package:flutter/material.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:timezone/data/latest.dart' as tz;
import 'package:timezone/timezone.dart' as tz;

void main() {
    runApp(ReminderApp());
}

class ReminderApp extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
        return MaterialApp(
                debugShowCheckedModeBanner: false,
                home: ReminderHome(),
    );
    }
}

class ReminderHome extends StatefulWidget {
    @override
    _ReminderHomeState createState() => _ReminderHomeState();
}

class _ReminderHomeState extends State<ReminderHome> {
    final FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin =
            FlutterLocalNotificationsPlugin();

    String _selectedDay = 'Monday';
    TimeOfDay _selectedTime = TimeOfDay.now();
    String _selectedActivity = 'Wake up';

    @override
    void initState() {
        super.initState();
        initializeNotifications();
    }

    void initializeNotifications() {
    const AndroidInitializationSettings initializationSettingsAndroid =
                AndroidInitializationSettings('@mipmap/ic_launcher');

    const InitializationSettings initializationSettings =
                InitializationSettings(android: initializationSettingsAndroid);

        flutterLocalNotificationsPlugin.initialize(initializationSettings);

        tz.initializeTimeZones();
    }

    Future<void> scheduleNotification() async {
        final time = Time(_selectedTime.hour, _selectedTime.minute, 0);

    const androidDetails = AndroidNotificationDetails(
                'reminder_channel_id',
                'Reminders',
                channelDescription: 'Channel for reminder notifications',
                importance: Importance.high,
                priority: Priority.high,
    );

    const notificationDetails = NotificationDetails(android: androidDetails);

        await flutterLocalNotificationsPlugin.zonedSchedule(
                0,
                'Reminder',
                '$_selectedActivity',
                _convertTimeToNextInstance(time),
                notificationDetails,
                androidAllowWhileIdle: true,
                uiLocalNotificationDateInterpretation:
        UILocalNotificationDateInterpretation.absoluteTime,
                matchDateTimeComponents: DateTimeComponents.time,
    );
    }

    tz.TZDateTime _convertTimeToNextInstance(Time time) {
        final now = tz.TZDateTime.now(tz.local);
        var scheduledDate = tz.TZDateTime(
                tz.local, now.year, now.month, now.day, time.hour, time.minute);
        if (scheduledDate.isBefore(now)) {
            scheduledDate = scheduledDate.add(const Duration(days: 1));
        }
        return scheduledDate;
    }

    @override
    Widget build(BuildContext context) {
        return Scaffold(
                appBar: AppBar(
                title: Text('Daily Reminder'),
      ),
        body: Padding(
                padding: const EdgeInsets.all(16.0)
                child: Column(
                children: [
        DropdownButton<String>(
                value: _selectedDay,
                onChanged: (newValue) {
                setState(() {
                _selectedDay = newValue!;
                });
              },
        items: <String>[
        'Monday',
                'Tuesday',
                'Wednesday',
                'Thursday',
                'Friday',
                'Saturday',
                'Sunday'
              ].map<DropdownMenuItem<String>>((String value) {
            return DropdownMenuItem<String>(
                    value: value,
                    child: Text(value),
                );
        }).toList(),
            ),
        SizedBox(height: 20),
        ElevatedButton(
                onPressed: () async {
            TimeOfDay? time = await showTimePicker(
                    context: context,
                    initialTime: _selectedTime,
                );
            if (time != null) {
                setState(() {
                    _selectedTime = time;
                });
            }
        },
        child: Text('Select Time'),
            ),
        SizedBox(height: 20),
        DropdownButton<String>(
                value: _selectedActivity,
                onChanged: (newValue) {
                setState(() {
                _selectedActivity = newValue!;
                });
              },
        items: <String>[
        'Wake up',
                'Go to gym',
                'Breakfast',
                'Meetings',
                'Lunch',
                'Quick nap',
                'Go to library',
                'Dinner',
                'Go to sleep',
              ].map<DropdownMenuItem<String>>((String value) {
            return DropdownMenuItem<String>(
                    value: value,
                    child: Text(value),
                );
        }).toList(),
            ),
        SizedBox(height: 40),
        ElevatedButton(
                onPressed: () {
            scheduleNotification();
        },
        child: Text('Set Reminder'),
            ),
          ],
        ),
      ),
    );
    }
}
