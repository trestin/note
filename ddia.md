# Designing Data-Intensive Application

## Part 1 数据系统的基石

### Chapter 1 可靠性、可扩展性、可维护性

应用程序：

- 数据密集型(data-intensive)
- 计算密集型(compute-intensive)

数据密集型应用程序的构成：

- 存储数据（数据库database）
- 缓存（cache）
- 流处理（stream processing）
- 批处理（batch processing）

软件系统重要问题：

- 可靠性（Reliability）
- 可扩展性（Scalability）
- 可维护性（Maintainability）

可靠性：

- 应用程序表现出用户所期望的功能。
- 允许用户犯错，允许用户以出乎意料的方式使用软件。
- 在预期的负载和数据量下，性能满足要求。
- 系统能防止未经授权的访问和滥用。